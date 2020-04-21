<template>
    <div :class="{custom_rx_input_container: true, readonly: readonly, active: active, error: !!error}"><div class="custom_input_title">Expression</div><div class="custom_rx_input">
        <div class="slash">/</div>
        <div class="editable_area" :contenteditable="!readonly" @input="update" @paste="onPaste" ref="editable" @focus="active = true" @blur="active = false"></div>
        <div class="slash">/</div>
        <div class="flags" ref="dropdownTrigger">{{flagsLocal.join('')}}
            <span class="ui-icon material-icons flags_icon">flag</span>
        </div>
        <or-popover trigger="dropdownTrigger" @open="onOpen" ref="dropdown" class="custom_rx_flag_menu">
            <div
                    class="menu_title"
                    style="text-align: center; padding: 5px 0; font-weight: bold;"
            >Regex Flags</div>
            <or-menu contain-focus has-icons has-secondary-text :options="menuOptions"
                     @close="onClose" @select="onSelect">
                <template slot-scope="props" slot="option">
                    <div class="ui-menu-option__content">
                        <div class="ui-menu-option__text">
                            {{props.option.label}}</div>
                        <div class="ui-menu-option__secondary-text" style="white-space: pre-line; position: relative; padding-right: 15px; text-align: right;">{{props.option.secondaryText}}<span
                                :class="{'ui-icon': true, 'ui-menu-option__icon': true, 'material-icons': true, [props.option.icon]: !!props.option.icon}"
                                v-if="props.option.showIcon"
                                style="position: absolute; margin-right: 0; right: -8px; top: calc(50% - 9px)"
                        >{{props.option.icon}}</span></div>
                    </div>
                </template>
            </or-menu>
        </or-popover>
    </div>
        <div class="error_text">{{error}}</div>
    </div>
</template>

<style>

</style>

<script>
	import Vue from 'vue';
	import orUi from '@onereach/ui';
	import 'normalize.css/normalize.css';
	import 'material-design-icons/iconfont/material-icons.css';
	import 'keen-ui/dist/keen-ui.css';
	import '@onereach/ui/dist/or-ui.css';

	Vue.use(orUi);

	export default {
		props: [
			'content',
			'readonly',
			'flags',
			'error',
			'htmlContent'
		],
		data() {
			return {
				flagsLocal: this.flags || [],
				active: false,
				cursorRange: [0, 0],
			}
		},
		mounted() {
			// this.$refs['editable'].innerHTML = this.content;
		},
		watch: {
			htmlContent(newVal, oldVal) {
				if (newVal !== oldVal) {
					this.$refs['editable'].innerHTML = this.htmlContent;
					this.setCaretPosition();
				}
			}
		},
		methods: {
			update() {
				this.cursorRange = this.getCaretPosition();
				let content = this.$refs['editable'].innerText;
				if (this.content === content.replace(/[\n\r\t\v]/g, '')) {
					this.$refs['editable'].innerHTML = this.htmlContent;
					this.setCaretPosition();
				} else {
					this.$emit('updateContent', content.replace(/[\n\r\t\v]/g, ''));
				}
			},
			onOpen() {
				if (this.readonly) {
					this.$refs['dropdown'].close();
				}
			},
			onClose() {
				this.$emit('onClose');
			},
			onSelect(e) {
				this.$emit('onSelect', e);
				let flagI = this.flagsLocal.indexOf(e.id);
				if (flagI === -1) {
					this.flagsLocal.push(e.id);
				} else {
					this.flagsLocal.splice(flagI, 1);
				}
				this.$emit('updateFlags', this.flagsLocal);
			},
			onPaste(e) {
				if (this.readonly) return;

				let paste = (e.clipboardData || window.clipboardData).getData('text');

				let pasteRx = paste.match(/^\/([^]+)\/(\w*?)$/);

				if (pasteRx) {
					const selection = window.getSelection();

					if (this.content === "" || this.content === selection.toString()) {
						e.preventDefault();
						let body = pasteRx[1].replace(/[\n\r\t\v]/g, '');
						this.cursorRange = [body.length, body.length];
						if (this.$refs['editable'].innerText === body) {
							this.$refs['editable'].innerHTML = this.htmlContent;
							this.setCaretPosition();
						}
						// this.$emit('updateContent', pasteRx[1]);
						let flags = pasteRx[2].replace(/[^gmuyis]/, '').split('').reduce((acc, flag) => {
							if (acc.indexOf(flag) === -1) {
								acc.push(flag);
							}
							return acc;
						}, []);
						this.flagsLocal = flags;
						this.$emit('updateFullRx', {body, flags});
					}
				} else if (this.$refs['editable'].innerText === paste.replace(/[\n\r\t\v]/g, '')) {
					this.cursorRange = [paste.length, paste.length];
					e.preventDefault();
					this.$refs['editable'].innerHTML = this.htmlContent;
					this.setCaretPosition();
				}
			},
			node_walk(node, func) {
				let result = func(node);
				for (node = node.firstChild; result !== false && node; node = node.nextSibling)
					result = this.node_walk(node, func);
				return result;
			},
			getCaretPosition() {
				let elem = this.$refs['editable'];
				let sel = window.getSelection();
				let cum_length = [0, 0];

				if (sel.anchorNode === elem)
					cum_length = [sel.anchorOffset, sel.extentOffset];
				else {
					let nodes_to_find = [sel.anchorNode, sel.extentNode];
					if (!elem.contains(sel.anchorNode) || !elem.contains(sel.extentNode))
						return undefined;
					else {
						let found = [0, 0];
						let i;
						this.node_walk(elem, function (node) {
							for (i = 0; i < 2; i++) {
								if (node === nodes_to_find[i]) {
									found[i] = true;
									if (found[i === 0 ? 1 : 0])
										return false; // all done
								}
							}

							if (node.textContent && !node.firstChild) {
								for (i = 0; i < 2; i++) {
									if (!found[i])
										cum_length[i] += node.textContent.length;
								}
							}
						});
						cum_length[0] += sel.anchorOffset;
						cum_length[1] += sel.extentOffset;
					}
				}
				if (cum_length[0] <= cum_length[1])
					return cum_length;
				return [cum_length[1], cum_length[0]];
			},
			getTextNodesIn(node) {
				let textNodes = [];
				if (node.nodeType === 3) {
					textNodes.push(node);
				} else {
					let children = node.childNodes;
					for (let i = 0, len = children.length; i < len; ++i) {
						textNodes.push.apply(textNodes, this.getTextNodesIn(children[i]));
					}
				}
				return textNodes;
			},
			setSelectionRange(el, start, end) {
				if (document.createRange && window.getSelection) {
					let range = document.createRange();
					range.selectNodeContents(el);
					let textNodes = this.getTextNodesIn(el);
					let foundStart = false;
					let charCount = 0, endCharCount;

					// eslint-disable-next-line
					for (let i = 0, textNode; textNode = textNodes[i++];) {
						endCharCount = charCount + textNode.length;
						if (!foundStart && start >= charCount
							&& (start < endCharCount ||
								(start === endCharCount && i <= textNodes.length))) {
							range.setStart(textNode, start - charCount);
							foundStart = true;
						}
						if (foundStart && end <= endCharCount) {
							range.setEnd(textNode, end - charCount);
							break;
						}
						charCount = endCharCount;
					}

					let sel = window.getSelection();
					sel.removeAllRanges();
					sel.addRange(range);
				} else if (document.selection && document.body.createTextRange) {
					let textRange = document.body.createTextRange();
					textRange.moveToElementText(el);
					textRange.collapse(true);
					textRange.moveEnd("character", end);
					textRange.moveStart("character", start);
					textRange.select();
				}
			},
			setCaretPosition() {
				let elem = this.$refs['editable'];

				this.setSelectionRange(elem, this.cursorRange[0], this.cursorRange[1]);
			}
		},
		computed: {
			menuOptions() {
				return [
					{
						"id": "g",
						"label": "g",
						"icon": "check",
						"showIcon": this.flags.indexOf('g') !== -1 ? "check" : undefined,
						"secondaryText": "Don't return after first match"
					},
					{
						"id": "m",
						"label": "m",
						"icon": "check",
						"showIcon": this.flags.indexOf('m') !== -1 ? "check" : undefined,
						"secondaryText": "^ and $ match start/end of line"
					},
					{
						"id": "i",
						"label": "i",
						"icon": "check",
						"showIcon": this.flags.indexOf('i') !== -1 ? "check" : undefined,
						"secondaryText": "Case insensitive match",
					},
					{
						"id": "u",
						"label": "u",
						"icon": "check",
						"showIcon": this.flags.indexOf('u') !== -1 ? "check" : undefined,
						"secondaryText": "Match with full unicode"
					},
					{
						"id": "s",
						"label": "s",
						"icon": "check",
						"showIcon": this.flags.indexOf('s') !== -1 ? "check" : undefined,
						"secondaryText": "Dot matches newline"
					},
					{
						"id": "y",
						"label": "y",
						"icon": "check",
						"showIcon": this.flags.indexOf('y') !== -1 ? "check" : undefined,
						"secondaryText": "Proceed matching from where\nprevious match ended only"
					}
				]
			}
		}
    }
</script>