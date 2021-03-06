<template>
    <div
            :class="classes"
            @mouseenter="handleMouseenter"
            @mouseleave="handleMouseleave"
            v-clickoutside="handleClose">
        <div
                :class="[prefixClass + '-rel']"
                ref="reference"
                @click="handleClick"
                @mousedown="handleFocus(false)"
                @mouseup="handleBlur(false)">
            <slot></slot>
        </div>
        <transition name="fade">
            <div
                    :class="popperClasses"
                    :style="styles"
                    ref="popper"
                    v-show="visible"
                    @click="handleTransferClick"
                    @mouseenter="handleMouseenter"
                    @mouseleave="handleMouseleave"
                    :data-transfer="transfer"
                    v-transfer-dom>
                <div :class="[prefixClass + '-content']">
                    <div :class="[prefixClass + '-arrow']"></div>
                    <div :class="[prefixClass + '-inner']" v-if="confirm">
                        <div :class="[prefixClass + '-body']">
                            <i class="icon-info"></i>
                            <div :class="[prefixClass + '-body-message']"><slot name="title">{{ title }}</slot></div>
                        </div>
                        <div :class="[prefixClass + '-footer']">
                            <i-button size="small" scene="danger" border circle @click.native="cancel">{{ localeCancelText }}</i-button>
                            <i-button size="small" scene="primary" border circle @click.native="ok">{{ localeOkText }}</i-button>
                        </div>
                    </div>
                    <div :class="[prefixClass + '-inner']" v-if="!confirm">
                        <div :class="[prefixClass + '-title']" v-if="showTitle" ref="title"><slot name="title"><div :class="[prefixClass + '-title-inner']">{{ title }}</div></slot></div>
                        <div :class="[prefixClass + '-body']">
                            <div :class="[prefixClass + '-body-content']"><slot name="content"><div :class="[prefixClass + '-body-content-inner']">{{ content }}</div></slot></div>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>
<script>
    import Popper from '../base/Popper';
    import iButton from '../button/Button.vue';
    import clickoutside from '../../directives/clickoutside';
    import TransferDom from '../../directives/transfer-dom';
    import { oneOf } from '../../utils/assist';
    import Locale from '../../mixins/locale';

    const prefixClass = 'poptip';

    export default {
        name: 'Poptip',
        mixins: [ Popper, Locale ],
        directives: { clickoutside, TransferDom },
        components: { iButton },
        props: {
            trigger: {
                validator (value) {
                    return oneOf(value, ['click', 'focus', 'hover']);
                },
                default: 'click'
            },
            placement: {
                validator (value) {
                    return oneOf(value, ['top', 'top-start', 'top-end', 'bottom', 'bottom-start', 'bottom-end', 'left', 'left-start', 'left-end', 'right', 'right-start', 'right-end']);
                },
                default: 'top'
            },
            title: {
                type: [String, Number]
            },
            content: {
                type: [String, Number],
                default: ''
            },
            width: {
                type: [String, Number]
            },
            confirm: {
                type: Boolean,
                default: false
            },
            okText: {
                type: String
            },
            cancelText: {
                type: String
            },
            transfer: {
                type: Boolean,
                default: false
            }
        },
        data () {
            return {
                prefixClass: prefixClass,
                showTitle: true,
                isInput: false,
                disableCloseUnderTransfer: false,  // transfer 模式下，点击 slot 也会触发关闭
            };
        },
        computed: {
            classes () {
                return [
                    `${prefixClass}`,
                    {
                        [`${prefixClass}-confirm`]: this.confirm
                    }
                ];
            },
            popperClasses () {
                return [
                    `${prefixClass}-popper`,
                    {
                        [`${prefixClass}-confirm`]: this.transfer && this.confirm
                    }
                ];
            },
            styles () {
                let style = {};

                if (this.width) {
                    style.width = `${this.width}px`;
                }
                return style;
            },
            localeOkText () {
                if (this.okText === undefined) {
                    return this.t('core.poptip.okText');
                }
                else {
                    return this.okText;
                }
            },
            localeCancelText () {
                if (this.cancelText === undefined) {
                    return this.t('core.poptip.cancelText');
                }
                else {
                    return this.cancelText;
                }
            }
        },
        methods: {
            handleClick () {
                if (this.confirm) {
                    this.visible = !this.visible;
                    return true;
                }
                if (this.trigger !== 'click') {
                    return false;
                }
                this.visible = !this.visible;
            },
            handleTransferClick () {
                if (this.transfer)
                    this.disableCloseUnderTransfer = true;
            },
            handleClose () {
                if (this.disableCloseUnderTransfer) {
                    this.disableCloseUnderTransfer = false;
                    return false;
                }
                if (this.confirm) {
                    this.visible = false;
                    return true;
                }
                if (this.trigger !== 'click') {
                    return false;
                }
                this.visible = false;
            },
            handleFocus (fromInput = true) {
                if (this.trigger !== 'focus' || this.confirm || (this.isInput && !fromInput)) {
                    return false;
                }
                this.visible = true;
            },
            handleBlur (fromInput = true) {
                if (this.trigger !== 'focus' || this.confirm || (this.isInput && !fromInput)) {
                    return false;
                }
                this.visible = false;
            },
            handleMouseenter () {
                if (this.trigger !== 'hover' || this.confirm) {
                    return false;
                }
                if (this.enterTimer) clearTimeout(this.enterTimer);
                this.enterTimer = setTimeout(() => {
                    this.visible = true;
                }, 100);
            },
            handleMouseleave () {
                if (this.trigger !== 'hover' || this.confirm) {
                    return false;
                }
                if (this.enterTimer) {
                    clearTimeout(this.enterTimer);
                    this.enterTimer = setTimeout(() => {
                        this.visible = false;
                    }, 100);
                }
            },
            cancel () {
                this.visible = false;
                this.$emit('on-cancel');
            },
            ok () {
                this.visible = false;
                this.$emit('on-ok');
            },
            getInputChildren () {
                const $input = this.$refs.reference.querySelectorAll('input');
                const $textarea = this.$refs.reference.querySelectorAll('textarea');
                let $children = null;

                if ($input.length) {
                    $children = $input[0];
                }
                else if ($textarea.length) {
                    $children = $textarea[0];
                }
                return $children;
            }
        },
        mounted () {
            if (!this.confirm) {
                this.showTitle = (this.$slots.title !== undefined) || this.title;
            }
            // if trigger and children is input or textarea,listen focus & blur event
            if (this.trigger === 'focus') {
                this.$nextTick(() => {
                    const $children = this.getInputChildren();
                    if ($children) {
                        this.isInput = true;
                        $children.addEventListener('focus', this.handleFocus, false);
                        $children.addEventListener('blur', this.handleBlur, false);
                    }
                });
            }
        },
        beforeDestroy () {
            const $children = this.getInputChildren();
            if ($children) {
                $children.removeEventListener('focus', this.handleFocus, false);
                $children.removeEventListener('blur', this.handleBlur, false);
            }
        }
    };
</script>
