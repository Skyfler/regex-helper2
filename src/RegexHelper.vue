<template>
    <div class="regex_helper">
        <custom-rx-input
                :content="regexBody"
                :htmlContent="htmlContent"
                :flags="flags"
                :readonly="readonly"
                :error="error"
                @updateContent="onUpdateContent"
                @updateFlags="onUpdateFlags"
                @updateFullRx="onUpdateFullRx"
                ref="customRxInput"
        ></custom-rx-input>

        <or-collapsible title="Reference">
            <div ref="reference_collaps" class="reference">
                <ul class="left_col" ref="left_col">
                    <li
                            @click="selectReferenceCategory(category)"
                            class="category_item"
                            v-for="category in referenceCategories"
                            v-bind:key="category"
                    >
                        <div class="label">{{ category }}</div>
                        <span class="ui-icon material-icons flags_keyboard_arrow_right">keyboard_arrow_right</span></li>
                </ul>
                <div :class="{right_col: true, active: !!currentCategory}" ref="right_col">
                    <ul>
                        <li class="reference_item" @click="resetReferenceCategory">
                            <div class="reference_item_title">
                                <span class="ui-icon material-icons flags_keyboard_arrow_left">keyboard_arrow_left</span>
                                <span>Back</span>
                            </div>
                        </li>
                        <li
                                v-for="refItem in currentReferenceItems"
                                :class="{reference_item: true, selected: refItem.name === currentReference}"
                                v-bind:key="refItem.name"
                        >
                            <div class="reference_item_title" @click="toggleRefDescription(refItem)">
                                <div class="ref_item_name">{{refItem.name}}</div>
                                <div class="ref_item_example">{{refItem.example}}</div>
                                <span v-if="refItem.name !== currentReference"
                                      class="ui-icon material-icons flags_keyboard_arrow_down">keyboard_arrow_down</span>
                                <span v-if="refItem.name === currentReference"
                                      class="ui-icon material-icons flags_keyboard_arrow_up">keyboard_arrow_up</span>
                            </div>
                            <div class="reference_item_description"
                                 :style="{height: refItem.name === currentReference ? currentDescriptionHeight + 'px' : 0}">
                                <div class="reference_item_description_inner" :ref="`reference_item_${refItem.name}`">
                                    <span class="ref_item_example">{{refItem.example}}</span>
                                    {{refItem.description}}
                                </div>
                            </div>
                        </li>
                    </ul>
                </div>
            </div>
        </or-collapsible>

        <or-collapsible title="Explanation" open>
            <div class="rx_expl" ref="expl_collaps"></div>
        </or-collapsible>

        <or-collapsible title="Test">
            <or-textbox
                    v-model="testStrValue"
                    :multi-line="true"
                    :readonly="readonly"
                    label="Test string"
            ></or-textbox>

            <or-tabs type="text" class="flex" fullwidth>
                <or-tab title="Match" :lazy="false">
                    <h3>Match result:</h3>
                    <div class="str_match_output" ref="str_match_output"></div>
                </or-tab>

                <or-tab title="Replace" :lazy="false">
                    <or-textbox
                            v-model="replaceStrValue"
                            :readonly="readonly"
                            label="Replace string"
                            :invalid="replaceStringError.length > 0"
                            :error="replaceStringError"
                    ></or-textbox>

                    <h3>Replace result:</h3>
                    <div class="str_replace_output" ref="str_replace_output"></div>
                </or-tab>
            </or-tabs>
        </or-collapsible>
    </div>
</template>

<style scoped lang="scss">
    .regex_helper {
        border: 1px solid #e4e4e4;
        padding: 10px;
        font-family: Roboto,-apple-system,BlinkMacSystemFont,Segoe UI,Oxygen,Ubuntu,Cantarell,Fira Sans,Droid Sans,Helvetica,Arial,sans-serif,Apple Color Emoji,Segoe UI Emoji,Segoe UI Symbol;
        font-size: 14px;
        -webkit-box-sizing: border-box;
        box-sizing: border-box;
        line-height: 1.5;
        text-align: left;

        ::v-deep *, &:after, &:before, :after, :before {
            -webkit-box-sizing: inherit;
            box-sizing: inherit;
        }

        ::v-deep .custom_rx_input_container {

            .custom_input_title {
                font-size: 12px;
                color: #91969d;
                line-height: 36px;
                -webkit-transform-origin: left;
                transform-origin: left;
                -webkit-transition: color .1s ease, -webkit-transform .2s ease;
                transition: color .1s ease, -webkit-transform .2s ease;
                transition: color .1s ease, transform .2s ease;
                transition: color .1s ease, transform .2s ease, -webkit-transform .2s ease;
            }

            .custom_rx_input {
                font-family: inherit;
                font-size: 14px;
                /*line-height: 1;*/
                line-height: normal;
                text-align: left;
                color: #0f232e;
                padding: 10px;
                min-height: 36px;
                border-radius: 3px;
                background-color: #f6f6f6;
                border: 1px solid #dfdfdf;
                -webkit-transition: border-color .5s ease-out, background-color .5s ease-out;
                transition: border-color .5s ease-out, background-color .5s ease-out;

                display: flex;
                flex-wrap: nowrap;

                .editable_area {
                    width: 100%;
                    white-space: nowrap;
                    overflow: hidden;
                    font-weight: bold;

                    &:focus {
                        outline: none;
                    }

                    .simple {
                        color: black;
                    }

                    .escaped_char {
                        color: #c0c;
                    }

                    .meta {
                        color: #d70;
                    }

                    .control {
                        color: #c0c
                    }

                    .hex {
                        color: #c0c
                    }

                    .decimal {
                        color: #c0c
                    }

                    .oct {
                        color: #c0c
                    }

                    .unicode {
                        color: #c0c
                    }

                    .assertion {
                        color: #840;
                    }

                    .quantifier {
                        color: #58f;
                    }

                    .char_class {
                        background: #fffab2;
                        color: #d70;
                    }

                    .char_class_range {

                    }

                    .unicode_prop {
                        color: #d70;
                    }

                    .disjunction {
                        color: #0a0;
                    }

                    .capture_group {
                        color: #0a0;
                        background: #bfeabf;
                    }

                    .backref {
                        color: #0a0;
                    }

                    .lookahead {
                        color: #0a0;
                        background: #bfeabf;
                    }

                    .lookbehind {
                        color: #0a0;
                        background: #bfeabf;
                    }

                    .error {
                        background: red;
                        color: #fff;
                    }
                }

                .slash {
                    padding: 0 2px;
                    color: darkgrey;
                    font-weight: bold;
                }

                .flags {
                    color: darkgrey;
                    white-space: nowrap;
                    position: relative;
                    padding-right: 19px;
                    font-weight: bold;
                    cursor: pointer;
                    -webkit-transition: color .5s ease-out;
                    transition: color .5s ease-out;

                    &:hover {
                        color: #505050;
                    }

                    .flags_icon {
                        font-size: 18px;
                        position: absolute;
                        right: 0;
                        top: 1px;
                    }
                }
            }

            .error_text {
                color: #f95d5d;
                visibility: hidden;
                font-size: 12px;
                line-height: 1.3;
            }

            &:not(.readonly) {
                &:hover {
                    .custom_rx_input {
                        background: transparent;
                        border-color: #a3a9b1;
                    }
                }

                &.active {
                    .custom_rx_input {
                        border-color: #64b2da;
                    }
                }
            }

            &.error {
                .custom_rx_input {
                    border-color: #f95d5d;
                }

                .error_text {
                    visibility: visible;
                }
            }

            &.readonly {
                .custom_rx_input {
                    color: rgba(0, 0, 0, .38);

                    .slash, .flags {
                        font-weight: normal;
                        color: rgba(0, 0, 0, .38);
                    }

                    .flags {
                        cursor: default;

                        &:hover {
                            color: rgba(0, 0, 0, .38);
                        }
                    }
                }
            }
        }

        ::v-deep .reference {
            /*display: flex;*/
            position: relative;
            overflow: hidden;

            .ui-icon {
                color: #a0a0a0;
            }

            .left_col {
                margin-bottom: 0;

                .category_item {
                    border: 1px solid #e4e4e4;
                    padding: 3px 10px;
                    cursor: pointer;
                    min-width: 135px;
                    display: flex;
                    justify-content: space-between;

                    & + .category_item {
                        border-top: 0;
                    }

                    &:hover {
                        background: #e4e4e4;
                    }

                    /*&.selected {*/
                    /*	font-weight: bold;*/
                    /*}*/
                }
            }

            .right_col {
                width: 100%;
                position: absolute;
                top: 0;
                left: 0;
                transform: translateX(100%);
                overflow-y: auto;
                height: 100%;
                background: #fff;
                transition: transform 0.3s ease;

                &.active {
                    left: 0;
                    transform: translateX(0%);
                }

                ul {
                    margin-bottom: 0;

                    .reference_item {

                        .reference_item_title {
                            display: flex;
                            align-items: center;
                            border: 1px solid #e4e4e4;
                            padding: 3px 0 3px 10px;
                            cursor: pointer;

                            &:hover {
                                background: #e4e4e4;
                            }

                            .ref_item_name {
                                width: 100%;
                            }

                            .ref_item_example {
                                white-space: nowrap;
                                font-weight: bold;
                                padding: 0 5px 0 5px;
                                color: #00126b;
                            }
                        }

                        .reference_item_description {
                            overflow: hidden;
                            border: 1px solid #e4e4e4;
                            border-top: 0;
                            background: #86868614;
                            height: 0;
                            -webkit-transition: height .2s ease;
                            -o-transition: height .2s ease;
                            transition: height .2s ease;
                            margin-top: -1px;

                            .reference_item_description_inner {
                                padding: 5px 10px 15px;

                                .ref_item_example {
                                    white-space: nowrap;
                                    font-weight: bold;
                                    padding-right: 5px;
                                    color: #00126b;
                                }
                            }
                        }

                        & + .reference_item {

                            .reference_item_title {
                                border-top: 0;
                            }
                        }
                    }
                }
            }
        }

        ::v-deep .rx_expl {
            text-align: left;

            .simple, .escaped_char, .meta, .control, .hex, .decimal, .oct, .unicode, .assertion, .char_class,
            .char_class_range, .unicode_prop, .capture_group, .lookahead, .lookbehind, .backref, .quantifier {
                padding: 2px 10px 2px 10px;
                border: solid 1px rgba(183, 188, 192, 0.5);
                border-left-width: 2px;
                margin-top: 4px;
            }

            .simple {
                background: rgb(246, 246, 246);

                & > .source {
                    color: black;
                }
            }

            .escaped_char {
                background: rgb(246, 246, 246);

                & > .source {
                    color: #c0c;
                }
            }

            .meta {
                background: rgb(246, 246, 246);

                & > .source {
                    color: #d70;
                }
            }

            .control {
                background: rgb(246, 246, 246);

                & > .source {
                    color: #c0c;
                }
            }

            .hex {
                background: rgb(246, 246, 246);

                & > .source {
                    color: #c0c;
                }
            }

            .decimal {
                background: rgb(246, 246, 246);

                & > .source {
                    color: #c0c;
                }
            }

            .oct {
                background: rgb(246, 246, 246);

                & > .source {
                    color: #c0c;
                }
            }

            .unicode {
                background: rgb(246, 246, 246);

                & > .source {
                    color: #c0c;
                }
            }

            .assertion {
                background: rgb(246, 246, 246);

                & > .source {
                    color: #840;
                }
            }

            .quantifier {
                margin-top: -1px;
                margin-left: 10px;
                background: rgba(183, 188, 192, .3);

                & > .source {
                    color: #58f;
                }
            }

            .char_class {
                background: #fffab2;
                border-color: #d70;

                & > .source {
                    color: #d70;
                }
            }

            .char_class_range {
                background: rgb(246, 246, 246);

                & > .source {
                    color: #d70;
                }
            }

            .unicode_prop {
                background: rgb(246, 246, 246);

                & > .source {
                    color: #d70;
                }
            }

            .disjunction {

            }

            .disjunction_symbol {
                background: rgb(246, 246, 246);

                & > .source {
                    color: #0a0;
                }
            }

            .capture_group {
                background: #bfeabf;
                border-color: #0a0;

                & > .source {
                    color: #0a0;
                }
            }

            .backref {
                background: rgb(246, 246, 246);

                & > .source {
                    color: #0a0;
                }
            }

            .lookahead {
                background: #bfeabf;
                border-color: #0a0;

                & > .source {
                    color: #0a0;
                }
            }

            .lookbehind {
                background: #bfeabf;
                border-color: #0a0;

                & > .source {
                    color: #0a0;
                }
            }

            .source {
                margin-right: 5px;

                .simple_source {
                    color: black;
                }

                .escaped_char_source {
                    color: #c0c;
                }

                .meta_source {
                    color: #d70;
                }

                .control_source {
                    color: #c0c
                }

                .hex_source {
                    color: #c0c
                }

                .decimal_source {
                    color: #c0c
                }

                .oct_source {
                    color: #c0c
                }

                .unicode_source {
                    color: #c0c
                }

                .assertion_source {
                    color: #840;
                }

                .quantifier_source {
                    color: #58f;
                }

                .char_class_source {
                    background: #fffab2;
                    color: #d70;
                }

                .char_class_range_source {

                }

                .unicode_prop_source {
                    color: #d70;
                }

                .disjunction_source {
                    color: #0a0;
                }

                .capture_group_source {
                    color: #0a0;
                    background: #bfeabf;
                }

                .backref_source {
                    color: #0a0;
                }

                .lookahead_source {
                    color: #0a0;
                    background: #bfeabf;
                }

                .lookbehind_source {
                    color: #0a0;
                    background: #bfeabf;
                }
            }

            .child {
                margin-left: 10px;
            }
        }

        ::v-deep .str_match_output {

            .match_container {
                background: #f6f6f6;
                border: 1px solid #d7d9db;
                width: 100%;
                table-layout: auto;
                border-collapse: separate;
                border-spacing: 0 5px;
                margin-bottom: 4px;

                td, th {
                    padding: 0px 10px;

                    & + td, & + th {
                        padding-left: 0;
                    }

                    &.match_label {
                        white-space: nowrap;
                        font-style: italic;
                    }

                    &.match_text {
                        width: 100%;
                    }
                }
            }
        }

        ::v-deep .str_replace_output {
            font-family: inherit;
            font-size: 14px;
            line-height: 1;
            text-align: left;
            color: #0f232e;
            padding: 10px;
            min-height: 36px;
            border-radius: 3px;
            background-color: #f6f6f6;
            border: 1px solid #dfdfdf;
            -webkit-transition: border-color .5s ease-out, background-color .5s ease-out;
            transition: border-color .5s ease-out, background-color .5s ease-out;
            white-space: pre-wrap;
        }
    }
</style>

<script>
	import Vue from 'vue';
	import orUi from '@onereach/ui';
	import 'normalize.css/normalize.css';
	import 'material-design-icons/iconfont/material-icons.css';
	import 'keen-ui/dist/keen-ui.css';
	import '@onereach/ui/dist/or-ui.css';
	import * as _ from 'lodash';
	import unraw from 'unraw';
	import regexpTree from 'regexp-tree';

	Vue.use(orUi);

	import CustomRxInput from './CustomRxInput.vue';

	export default {
		props: {
			value: {
				type: String,
				default: ''
			},
			readonly: {
				type: Boolean,
				default: false
			}
		},

		data() {
			return {
				testStrValue: '',
				replaceStrValue: '$&',
				rxNamedGroups: [],
				error: '',
				replaceStringError: '',
				coloredRx: '',
				htmlContent: '',
				types: {
					RegExp: {
						variants: [{
							condition() {
								return true;
							},
							fn: (node) => {
								this.coloredRx = '';
								const res = this.getVariant(node.body).fn(node.body);
								return res;
							}
						}],
						childPropName: 'body'
					},
					Char: {
						variants: [
							{
								condition(node) {
									return node.kind === 'simple' && !node.escaped
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="simple_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="simple">${_.escape(node.loc.source)}</span>`;
									return `<div class="simple"><b class="source">${_.escape(node.loc.source)}</b> matches the character ${_.escape(node.symbol)} literally (${this.flags.indexOf('i') !== -1 ? 'case insensitive' : 'case sensitive'})</div>`
								}
							},
							{
								condition(node) {
									return node.kind === 'simple' && node.escaped
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="escaped_char_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="escaped_char">${_.escape(node.loc.source)}</span>`;
									return `<div class="escaped_char"><b class="source">${_.escape(node.loc.source)}</b> matches the character ${_.escape(node.symbol)} literally (${this.flags.indexOf('i') !== -1 ? 'case insensitive' : 'case sensitive'})</div>`
								}
							},
							{
								condition(node) {
									return node.kind === 'meta'
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="meta_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="meta">${_.escape(node.loc.source)}</span>`;
									switch (node.value) {
										case '.':
											return `<div class="meta"><b class="source">${_.escape(node.loc.source)}</b> matches any character (${this.flags.indexOf('s') !== -1 ? ' (except for line terminators)' : ''})</div>`
										case '\\f':
											return `<div class="meta"><b class="source">${_.escape(node.loc.source)}</b> matches a form-feed character</div>`
										case '\\r':
											return `<div class="meta"><b class="source">${_.escape(node.loc.source)}</b> matches a carriage return</div>`
										case '\\n':
											return `<div class="meta"><b class="source">${_.escape(node.loc.source)}</b> matches a line-feed (newline) character</div>`
										case '\\t':
											return `<div class="meta"><b class="source">${_.escape(node.loc.source)}</b> matches a tab character</div>`
										case '\\v':
											return `<div class="meta"><b class="source">${_.escape(node.loc.source)}</b> matches a vertical tab character</div>`
										case '\\0':
											return `<div class="meta"><b class="source">${_.escape(node.loc.source)}</b> matches NULL character</div>`
										case '\\b':
											return `<div class="meta"><b class="source">${_.escape(node.loc.source)}</b> matches a backspace character</div>`
										case '\\s':
											return `<div class="meta"><b class="source">${_.escape(node.loc.source)}</b> matches any whitespace character (equal to [\\r\\n\\t\\f\\v ])</div>`
										case '\\S':
											return `<div class="meta"><b class="source">${_.escape(node.loc.source)}</b> matches any non-whitespace character (equal to [^\\r\\n\\t\\f\\v ])</div>`
										case '\\w':
											return `<div class="meta"><b class="source">${_.escape(node.loc.source)}</b> matches any word character (equal to [a-zA-Z0-9_])</div>`
										case '\\W':
											return `<div class="meta"><b class="source">${_.escape(node.loc.source)}</b> matches any non-word character (equal to [^a-zA-Z0-9_])</div>`
										case '\\d':
											return `<div class="meta"><b class="source">${_.escape(node.loc.source)}</b> matches a digit (equal to [0-9])</div>`
										case '\\D':
											return `<div class="meta"><b class="source">${_.escape(node.loc.source)}</b> matches any character that's not a digit (equal to [^0-9])</div>`
									}
								}
							},
							{
								condition(node) {
									return node.kind === 'control'
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="control_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="control">${_.escape(node.loc.source)}</span>`;
									return `<div class="control"><b class="source">${_.escape(node.loc.source)}</b> matches the control sequence CTRL+${node.value.substring(node.value.length - 1)}</div>`
								}
							},
							{
								condition(node) {
									return node.kind === 'hex'
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="hex_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="hex">${_.escape(node.loc.source)}</span>`;
									return `<div class="hex"><b class="source">${_.escape(node.loc.source)}</b> matches the character ${_.escape(node.symbol)} (${this.flags.indexOf('i') !== -1 ? 'case insensitive' : 'case sensitive'})</div>`
								}
							},
							{
								condition(node) {
									return node.kind === 'decimal' && RegExp(node.value).test(node.symbol)
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="decimal_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="decimal">${_.escape(node.loc.source)}</span>`;
									return `<div class="decimal"><b class="source">${_.escape(node.loc.source)}</b> matches the character ${_.escape(node.symbol)} (${this.flags.indexOf('i') !== -1 ? 'case insensitive' : 'case sensitive'})</div>`
								}
							},
							{
								condition(node) {
									return node.kind === 'oct' || (node.kind === 'decimal' && !RegExp(node.value).test(node.symbol))
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									let symbol = node.symbol;
									if (node.kind === 'decimal' && !RegExp(node.value).test(node.symbol)) {
										symbol = String.fromCharCode(parseInt(node.value.substring(1), 8));
									}
									parentSourceObj.source += `<span class="oct_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="oct">${_.escape(node.loc.source)}</span>`;
									return `<div class="oct"><b class="source">${_.escape(node.loc.source)}</b> matches the character ${_.escape(symbol)} (${this.flags.indexOf('i') !== -1 ? 'case insensitive' : 'case sensitive'})</div>`
								}
							},
							{
								condition(node) {
									return node.kind === 'unicode' && !node.isSurrogatePair
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="unicode_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="unicode">${_.escape(node.loc.source)}</span>`;
									return `<div class="unicode"><b class="source">${_.escape(node.loc.source)}</b> matches the character ${_.escape(node.symbol)} (${this.flags.indexOf('i') !== -1 ? 'case insensitive' : 'case sensitive'})</div>`
								}
							},
							{
								condition(node) {
									return node.kind === 'unicode' && node.isSurrogatePair
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="unicode_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="unicode">${_.escape(node.loc.source)}</span>`;
									return `<div class="unicode"><b class="source">${_.escape(node.loc.source)}</b> matches the character ${_.escape(node.symbol)} (${this.flags.indexOf('i') !== -1 ? 'case insensitive' : 'case sensitive'})</div>`
								}
							}
						]
					},
					CharacterClass: {
						variants: [
							{
								condition(node) {
									return !node.negative
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									let localSource = {source: ''};
									localSource.source += `<span class="char_class_source">[`;
									this.coloredRx += `<span class="char_class">[`;
									let children = node.expressions.length ? node.expressions.map(childNode => {
										if (childNode.type === 'Backreference') {
											childNode = this.convertBackrefToSymb(childNode);
										}

										return this.getVariant(childNode).fn(childNode, localSource);
									}) : [];
									localSource.source += `]</span>`;
									this.coloredRx += `]</span>`;
									const res = children.length ?
										`<div class="char_class"><b class="source">${localSource.source/*_.escape(node.loc.source)*/}</b> Matches a single character present in the list below:` +
										`<div class="child">${children.join('')}</div></div>` :
										`<div class="char_class"><b class="source">${localSource.source/*_.escape(node.loc.source)*/}</b> Empty character class - matches <i>null</i></div>`
									parentSourceObj.source += localSource.source;
									return res;
								}
							},
							{
								condition(node) {
									return node.negative
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									let localSource = {source: ''};
									localSource.source += `<span class="char_class_source">[^`;
									this.coloredRx += `<span class="char_class">[^`;
									let children = node.expressions.length ? node.expressions.map(childNode => this.getVariant(childNode).fn(childNode, localSource)) : [];
									localSource.source += `]</span>`;
									this.coloredRx += `]</span>`;
									const res = children.length ?
										`<div class="char_class"><b class="source">${localSource.source/*_.escape(node.loc.source)*/}</b> Matches a single character not present in the list below:` +
										`<div class="child">${children.join('')}</div></div>` :
										`<div class="char_class"><b class="source">${localSource.source/*_.escape(node.loc.source)*/}</b> Matches any character, including newline</div>`
									parentSourceObj.source += localSource.source;
									return res;
								}
							},
						],
						childPropName: 'expressions'
					},
					ClassRange: {
						variants: [{
							condition() {
								return true
							},
							fn: (node, parentSourceObj = {source: ''}) => {
								let localSource = {source: ''};
								let nodeFrom = node.from;
								let nodeTo = node.to;
								if (nodeFrom.type === 'Backreference') {
									nodeFrom = this.convertBackrefToSymb(nodeFrom);
								}
								if (nodeTo.type === 'Backreference') {
									nodeTo = this.convertBackrefToSymb(nodeTo);
								}
								let fromKind = nodeFrom.kind === 'simple' && nodeFrom.escaped ? 'escaped_char' : nodeFrom.kind;
								let toKind = nodeTo.kind === 'simple' && nodeTo.escaped ? 'escaped_char' : nodeTo.kind;
								localSource.source = `<span class="char_class_range_source"><span class="${fromKind}_source">${fromKind === 'escaped_char' ? '\\' : ''}${_.escape(nodeFrom.value)}</span>-<span class="${toKind}_source">${toKind === 'escaped_char' ? '\\' : ''}${_.escape(nodeTo.value)}</span></span>`;
								this.coloredRx += `<span class="char_class_range"><span class="${fromKind}">${fromKind === 'escaped_char' ? '\\' : ''}${_.escape(nodeFrom.value)}</span>-<span class="${toKind}">${toKind === 'escaped_char' ? '\\' : ''}${_.escape(nodeTo.value)}</span></span>`;
								parentSourceObj.source += localSource.source;
								return `<div class="char_class_range"><b class="source">${localSource.source/*_.escape(node.loc.source)*/}</b> a single character in the range between ${_.escape(nodeFrom.symbol ? nodeFrom.symbol : nodeFrom.value)} and ${_.escape(nodeTo.symbol ? nodeTo.symbol : nodeTo.value)} (${this.flags.indexOf('i') !== -1 ? 'case insensitive' : 'case sensitive'})</div>`
							}
						}],
					},
					UnicodeProperty: {
						variants: [
							{
								condition(node) {
									return !node.negative
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="unicode_prop_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="unicode_prop">${_.escape(node.loc.source)}</span>`;
									return `<div class="unicode_prop"><b class="source">${_.escape(node.loc.source)}</b> matches a single character of the "${_.escape(node.canonicalValue)}" unicode category</div>`
								}
							},
							{
								condition(node) {
									return node.negative
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="unicode_prop_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="unicode_prop">${_.escape(node.loc.source)}</span>`;
									return `<div class="unicode_prop"><b class="source">${_.escape(node.loc.source)}</b> matches a single character of the "${_.escape(node.canonicalValue)}" unicode category</div>`
								}
							}
						],
					},
					Alternative: {
						variants: [{
							condition() {
								return true
							},
							fn: (node, parentSourceObj = {source: ''}) => {
								let simpleChars = [];

								let res = node.expressions.reduce((acc, childNode) => {
									if (childNode.type === 'Char' && childNode.kind === 'simple' && !childNode.escaped) {
										simpleChars.push(_.escape(childNode.loc.source));
									} else {
										if (simpleChars.length) {
											acc.push(`<div class="simple"><b class="source">${simpleChars.join('')}</b> matches the character${simpleChars.length > 1 ? 's' : ''} ${simpleChars.join('')} literally (${this.flags.indexOf('i') !== -1 ? 'case insensitive' : 'case sensitive'})</div>`);
											parentSourceObj.source += `<span class="simple_source">${simpleChars.join('')}</span>`;
											this.coloredRx += `<span class="simple">${simpleChars.join('')}</span>`;
											simpleChars = [];
										}
										acc.push(this.getVariant(childNode).fn(childNode, parentSourceObj));
									}
									return acc;
								}, []);

								if (simpleChars.length) {
									res.push(`<div class="simple"><b class="source">${simpleChars.join('')}</b> matches the character${simpleChars.length > 1 ? 's' : ''} ${simpleChars.join('')} literally (${this.flags.indexOf('i') !== -1 ? 'case insensitive' : 'case sensitive'})</div>`);
									parentSourceObj.source += `<span class="simple_source">${simpleChars.join('')}</span>`;
									this.coloredRx += `<span class="simple">${simpleChars.join('')}</span>`;
								}

								return res.join('');
							}
						}],
						childPropName: 'expressions'
					},
					Disjunction: {
						variants: [{
							condition() {
								return true
							},
							fn: (node, parentSourceObj = {source: ''}) => {
								let left;
								let right;
								if (node.left) {
									left = node.left.type === 'Disjunction' ? this.getVariant(node.left).fn(node.left, parentSourceObj) : [this.getVariant(node.left).fn(node.left, parentSourceObj)];
								} else {
									left = ['<div class="simple"><i>null</i>, matches any position</div>'];
								}
								parentSourceObj.source += `<span class="disjunction_source">|</span>`;
								this.coloredRx += `<span class="disjunction">|</span>`;
								if (node.right) {
									right = node.right.type === 'Disjunction' ? this.getVariant(node.right).fn(node.right, parentSourceObj) : [this.getVariant(node.right).fn(node.right, parentSourceObj)];
								} else {
									right = ['<div class="simple"><i>null</i>, matches any position</div>'];
								}
								let arr = [
									...left,
									'<div class="simple disjunction_symbol"><b class="source">|</b> Alternation. Matches the expression before or after the |</div>',
									...right
								];
								return `<div class="disjunction">${arr.reduce((acc, elem) => acc + elem, '')}</div>`;
							}
						}],
					},
					Group: {
						variants: [
							{
								condition(node) {
									return node.capturing && !node.name && !node.nameRaw
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									let localSource = {source: ''};
									localSource.source += `<span class="capture_group_source">(`;
									this.coloredRx += `<span class="capture_group">(`;
									let children = node.expression ?
										this.getVariant(node.expression).fn(node.expression, localSource) :
										'<div class="simple"><i>null</i>, matches any position</div>';
									localSource.source += `)</span>`;
									this.coloredRx += `)</span>`;
									const res = `<div class="capture_group"><b class="source">${localSource.source/*_.escape(node.loc.source)*/}</b> Capture Group #${node.number}:` +
										`<div class="child">${children}</div></div>`;
									parentSourceObj.source += localSource.source;
									return res;
								}
							},
							{
								condition(node) {
									return node.capturing && typeof node.name === 'string' && typeof node.nameRaw === 'string'
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									this.rxNamedGroups.push({
										number: node.number,
										name: node.name
									});
									let localSource = {source: ''};
									localSource.source += `<span class="capture_group_source">(?${_.escape(`<${node.name}>`)}`;
									this.coloredRx += `<span class="capture_group">(?${_.escape(`<${node.name}>`)}`;
									let children = node.expression ?
										this.getVariant(node.expression).fn(node.expression, localSource) :
										'<div class="simple"><i>null</i>, matches any position</div>';
									localSource.source += `)</span>`;
									this.coloredRx += `)</span>`;
									const res = `<div class="capture_group"><b class="source">${localSource.source/*_.escape(node.loc.source)*/}</b> Named Capture Group "${_.escape(node.name)}":` +
										`<div class="child">${children}</div></div>`;
									parentSourceObj.source += localSource.source;
									return res;
								}
							},
							{
								condition(node) {
									return !node.capturing
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									let localSource = {source: ''};
									localSource.source += `<span class="capture_group_source">(?:`;
									this.coloredRx += `<span class="capture_group">(?:`;
									let children = node.expression ?
										this.getVariant(node.expression).fn(node.expression, localSource) :
										'<div class="simple"><i>null</i>, matches any position</div>';
									localSource.source += `)</span>`;
									this.coloredRx += `)</span>`;
									const res = `<div class="capture_group"><b class="source">${localSource.source/*_.escape(node.loc.source)*/}</b> Non-capturing group:` +
										`<div class="child">${children}</div></div>`;
									parentSourceObj.source += localSource.source;
									return res;
								}
							}
						],
						childPropName: 'expression'
					},
					Backreference: {
						variants: [
							{
								condition(node) {
									return typeof node.reference === 'number'
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="backref_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="backref">${_.escape(node.loc.source)}</span>`;
									return `<div class="backref"><b class="source">${_.escape(node.loc.source)}</b> Numeric reference. Matches the results of capture group #${_.escape(node.reference)}.</div>`
								}
							},
							{
								condition(node) {
									return typeof node.reference === 'string' && typeof node.referenceRaw === 'string'
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="backref_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="backref">${_.escape(node.loc.source)}</span>`;
									return `<div class="backref"><b class="source">${_.escape(node.loc.source)}</b> Named reference. Matches the results of capture group #${_.escape(node.reference)}.</div>`
								}
							}
						],
					},
					Repetition: {
						variants: [{
							condition() {
								return true
							},
							fn: (node, parentSourceObj = {source: ''}) => {
								return this.getVariant(node.expression).fn(node.expression, parentSourceObj) + this.getVariant(node.quantifier).fn(node.quantifier, parentSourceObj)
							}
						}],
						childPropName: 'expression'
					},
					Quantifier: {
						variants: [
							{
								condition(node) {
									return node.kind === '?'
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="quantifier_source">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</span>`;
									this.coloredRx += `<span class="quantifier">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</span>`;
									return `<div class="quantifier"><b class="source">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</b> quantifier, matches the preceding token between 0 and 1 times${!node.greedy ? ' as few times as possible (lazy)' : ''}</div>`
								}
							},
							{
								condition(node) {
									return node.kind === '*'
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="quantifier_source">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</span>`;
									this.coloredRx += `<span class="quantifier">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</span>`;
									return `<div class="quantifier"><b class="source">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</b> quantifier, matches the preceding token 0 or more times${!node.greedy ? ' as few times as possible (lazy)' : ''}</div>`
								}
							},
							{
								condition(node) {
									return node.kind === '+'
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="quantifier_source">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</span>`;
									this.coloredRx += `<span class="quantifier">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</span>`;
									return `<div class="quantifier"><b class="source">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</b> quantifier, matches the preceding token 1 or more times${!node.greedy ? ' as few times as possible (lazy)' : ''}</div>`
								}
							},
							{
								condition(node) {
									return node.kind === 'Range' && typeof node.from === 'number' && typeof node.to === 'number' && node.from === node.to
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="quantifier_source">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</span>`;
									this.coloredRx += `<span class="quantifier">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</span>`;
									return `<div class="quantifier"><b class="source">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</b> quantifier, matches the preceding token exactly ${_.escape(node.from)} times${!node.greedy ? ' as few times as possible (lazy)' : ''}</div>`
								}
							},
							{
								condition(node) {
									return node.kind === 'Range' && typeof node.from === 'number' && node.to === undefined
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="quantifier_source">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</span>`;
									this.coloredRx += `<span class="quantifier">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</span>`;
									return `<div class="quantifier"><b class="source">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</b> quantifier, matches the preceding token ${_.escape(node.from)} or more times${!node.greedy ? ' as few times as possible (lazy)' : ''}</div>`
								}
							},
							{
								condition(node) {
									return node.kind === 'Range' && typeof node.from === 'number' && typeof node.to === 'number' && node.from !== node.to
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="quantifier_source">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</span>`;
									this.coloredRx += `<span class="quantifier">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</span>`;
									return `<div class="quantifier"><b class="source">${_.escape(node.loc.source)}${!node.greedy ? '?' : ''}</b> quantifier, matches the preceding token between ${_.escape(node.from)} and ${_.escape(node.to)} times${!node.greedy ? ' as few times as possible (lazy)' : ''}</div>`
								}
							},
						],
					},
					Assertion: {
						variants: [
							{
								condition(node) {
									return node.kind === '^'
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="assertion_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="assertion">${_.escape(node.loc.source)}</span>`;
									return `<div class="assertion"><b class="source">${_.escape(node.loc.source)}</b> asserts position at start of the ${this.flags.indexOf('m') !== -1 ? 'line' : 'string'}</div>`
								}
							},
							{
								condition(node) {
									return node.kind === '$'
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="assertion_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="assertion">${_.escape(node.loc.source)}</span>`;
									return `<div class="assertion"><b class="source">${_.escape(node.loc.source)}</b> asserts position at end of the ${this.flags.indexOf('m') !== -1 ? 'line' : 'string'}</div>`
								}
							},
							{
								condition(node) {
									return node.kind === '\\b'
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="assertion_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="assertion">${_.escape(node.loc.source)}</span>`;
									return `<div class="assertion"><b class="source">${_.escape(node.loc.source)}</b> asserts position at a word boundary: (^\\w|\\w$|\\W\\w|\\w\\W)</div>`
								}
							},
							{
								condition(node) {
									return node.kind === '\\B'
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									parentSourceObj.source += `<span class="assertion_source">${_.escape(node.loc.source)}</span>`;
									this.coloredRx += `<span class="assertion">${_.escape(node.loc.source)}</span>`;
									return `<div class="assertion"><b class="source">${_.escape(node.loc.source)}</b> matches any position that is not a word boundary.</div>`
								}
							},
							{
								condition(node) {
									return node.kind === 'Lookahead' && !node.negative
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									let localSource = {source: ''};
									localSource.source += `<span class="lookahead_source">(?=`;
									this.coloredRx += `<span class="lookahead">(?=`;
									let children = node.assertion ?
										this.getVariant(node.assertion).fn(node.assertion, localSource) :
										'<div class="simple"><i>null</i>, matches any position</div>';
									localSource.source += `)</span>`;
									this.coloredRx += `)</span>`;
									const res = `<div class="lookahead"><b class="source">${localSource.source/*_.escape(node.loc.source)*/}</b> Positive Lookahead. Matches a group after the main expression without including it in the result.` +
										`<div class="child">${children}</div></div>`;
									parentSourceObj.source += localSource.source;
									return res;
								}
							},
							{
								condition(node) {
									return node.kind === 'Lookahead' && node.negative
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									let localSource = {source: ''};
									localSource.source += `<span class="lookahead_source">(?!`;
									this.coloredRx += `<span class="lookahead">(?!`;
									let children = node.assertion ?
										this.getVariant(node.assertion).fn(node.assertion, localSource) :
										'<div class="simple"><i>null</i>, matches any position</div>';
									localSource.source += `)</span>`;
									this.coloredRx += `)</span>`;
									const res = `<div class="lookahead"><b class="source">${localSource.source/*_.escape(node.loc.source)*/}</b> Negative Lookahead. Specifies a group that can not match after the main expression (if it matches, the result is discarded).` +
										`<div class="child">${children}</div></div>`;
									parentSourceObj.source += localSource.source;
									return res;
								}
							},
							{
								condition(node) {
									return node.kind === 'Lookbehind' && !node.negative
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									let localSource = {source: ''};
									localSource.source += `<span class="lookbehind_source">${_.escape('(?<=')}`;
									this.coloredRx += `<span class="lookbehind">${_.escape('(?<=')}`;
									let children = node.assertion ?
										this.getVariant(node.assertion).fn(node.assertion, localSource) :
										'<div class="simple"><i>null</i>, matches any position</div>';
									localSource.source += `)</span>`;
									this.coloredRx += `)</span>`;
									const res = `<div class="lookbehind"><b class="source">${_.escape(node.loc.source)}</b> Positive Lookbehind. Matches a group before the main expression without including it in the result.` +
										`<div class="child">${children}</div></div>`;
									parentSourceObj.source += localSource.source;
									return res;
								}
							},
							{
								condition(node) {
									return node.kind === 'Lookbehind' && node.negative
								},
								fn: (node, parentSourceObj = {source: ''}) => {
									let localSource = {source: ''};
									localSource.source += `<span class="lookbehind_source">${_.escape('(?<!')}`;
									this.coloredRx += `<span class="lookbehind">${_.escape('(?<!')}`;
									let children = node.assertion ?
										this.getVariant(node.assertion).fn(node.assertion, localSource) :
										'<div class="simple"><i>null</i>, matches any position</div>';
									localSource.source += `)</span>`;
									this.coloredRx += `)</span>`;
									const res = `<div class="lookbehind"><b class="source">${_.escape(node.loc.source)}</b> Negative Lookbehind. Specifies a group that can not match before the main expression (if it matches, the result is discarded).` +
										`<div class="child">${children}</div></div>`;
									parentSourceObj.source += localSource.source;
									return res;
								}
							},
						],
						childPropName: 'assertion'
					}
				},
				currentCategory: '',
				currentReference: '',
				currentDescriptionHeight: '',
				referenceCategories: [
					'All tokens',
					'Common tokens',
					'General tokens',
					'Anchors',
					'Meta sequences',
					'Quantifiers',
					'Group constructs',
					'Character classes',
					'Flags/Modifiers',
					'Substitution',
				],
				referenceItems: [
					{
						name: 'Newline',
						example: '\\n',
						description: 'Matches a newline character',
						categories: ['All tokens', 'General tokens', 'Substitution'],
					},
					{
						name: 'Carriage return',
						example: '\\r',
						description: 'Matches a carriage return character, unicode character 2185.',
						categories: ['All tokens', 'General tokens', 'Substitution'],
					},
					{
						name: 'Tab',
						example: '\\t ',
						description: 'Matches a tab character. Historically, tab stops happen every 8 characters.',
						categories: ['All tokens', 'General tokens', 'Substitution'],
					},
					{
						name: 'Null character',
						example: '\\0',
						description: 'Matches a null character, most often visually represented in unicode using U+2400.',
						categories: ['All tokens', 'General tokens'],
					},
					{
						name: 'Character set',
						example: '[abc]',
						description: 'Matches any character in the set.',
						categories: ['All tokens', 'Common tokens', 'Character classes',],
					},
					{
						name: 'Negated set',
						example: '[^abc]',
						description: 'Match any character that is not in the set.',
						categories: ['All tokens', 'Common tokens', 'Character classes',],
					},
					{
						name: 'Range',
						example: '[a-z]',
						description: 'Matches a character having a character code between the two specified characters inclusive.',
						categories: ['All tokens', 'Common tokens', 'Character classes',],
					},
					{
						name: 'Any single character',
						example: '.',
						description: 'Matches any character except linebreaks. Equivalent to [^\\n\\r].',
						categories: ['All tokens', 'Common tokens', 'Meta sequences',],
					},
					{
						name: 'Any whitespace character',
						example: '\\s',
						description: 'Matches any space, tab or newline character.',
						categories: ['All tokens', 'Common tokens', 'Meta sequences',],
					},
					{
						name: 'Any non-whitespace character',
						example: '\\S',
						description: 'Matches anything other than a space, tab or newline.',
						categories: ['All tokens', 'Common tokens', 'Meta sequences',],
					},
					{
						name: 'Any digit',
						example: '\\d ',
						description: 'Matches any decimal digit. Equivalent to [0-9].',
						categories: ['All tokens', 'Common tokens', 'Meta sequences',],
					},
					{
						name: 'Any non-digit',
						example: '\\D ',
						description: 'Matches anything other than a decimal digit.',
						categories: ['All tokens', 'Common tokens', 'Meta sequences',],
					},
					{
						name: 'Any word character',
						example: '\\w',
						description: 'Matches any letter, digit or underscore. Equivalent to [a-zA-Z0-9_].',
						categories: ['All tokens', 'Common tokens', 'Meta sequences',],
					},
					{
						name: 'Any non-word character',
						example: '\\W',
						description: 'Matches anything other than a letter, digit or underscore.',
						categories: ['All tokens', 'Common tokens', 'Meta sequences',],
					},
					{
						name: 'Vertical whitespace',
						example: '\\v',
						description: 'Matches newlines and vertical tabs. Works with Unicode. Vertical tabs can be inserted in some word processors using CMD/CTRL+ENTER.',
						categories: ['All tokens', 'Meta sequences',],
					},
					{
						name: 'Match nth subpattern',
						example: '\\n',
						description: 'Usually referred to as a `backreference`, this will match a repeat of the text captured in a previous set of parentheses. To reduce ambiguity one may also use `\\gn`, or `\\g{n}` where n is a digit.',
						categories: ['All tokens', 'Meta sequences',],
					},
					{
						name: 'Match subpattern `name`',
						example: '\\k<name>',
						description: 'Matches the text matched by a previously named capture group.',
						categories: ['All tokens', 'Meta sequences',],
					},
					{
						name: 'Hex character YYYY',
						example: '\\uYYYY',
						description: 'Matches the unicode character with the given hex value.',
						categories: ['All tokens', 'Meta sequences', 'Substitution'],
					},
					{
						name: 'Extended unicode character YYYY',
						example: '\\u{YYYY}',
						description: 'Matches the unicode character with the given hex value. Requires the unicode flag (u)',
						categories: ['All tokens', 'Meta sequences', 'Substitution'],
					},
					{
						name: 'Hex character YY',
						example: '\\xYY',
						description: 'Matches the 8-bit character with the given hex value.',
						categories: ['All tokens', 'Meta sequences', 'Substitution'],
					},
					{
						name: 'Octal chracter ddd',
						example: '\\ddd',
						description: 'Matches the 8-bit character with the given octal value.',
						categories: ['All tokens', 'Meta sequences', 'Substitution'],
					},
					{
						name: 'Control character Y',
						example: '\\cY',
						description: 'Matches ASCII characters typically associated with Control+A through Control+Z: \\x01 through \\x1A.',
						categories: ['All tokens', 'Meta sequences',],
					},
					{
						name: 'Backspace character',
						example: '[\\b]',
						description: 'Matches the backspace control character.',
						categories: ['All tokens', 'Meta sequences',],
					},
					{
						name: 'Form feed character',
						example: '\\f',
						description: 'Matches a FORM FEED character (char code 12).',
						categories: ['All tokens', 'Meta sequences', 'Substitution'],
					},
					{
						name: 'Makes any character literal',
						example: '\\',
						description: 'This may be used to match the literal value of any metacharacter, or the / delimiter.',
						categories: ['All tokens', 'Meta sequences',],
					},
					{
						name: 'Capturing group',
						example: '(...)',
						description: 'Groups multiple tokens together and creates a capture group for extracting a substring or using a backreference.',
						categories: ['All tokens', 'Common tokens', 'Group constructs',],
					},
					{
						name: 'Alteration',
						example: 'a|b',
						description: 'Matches the a or the b part of the subexpression',
						categories: ['All tokens', 'Common tokens', 'Group constructs',],
					},
					{
						name: 'Non-capturing group',
						example: '(?:...)',
						description: 'This construct is similar to (...), but won\'t create a capture group.',
						categories: ['All tokens', 'Group constructs',],
					},
					{
						name: 'Named capturing group',
						example: '(?<name>...)',
						description: 'This capturing group can be referred to using the given name instead of a number.',
						categories: ['All tokens', 'Group constructs',],
					},
					{
						name: 'Positive lookahead',
						example: '(?=...)',
						description: 'Asserts that the given subpattern can be matched here, without consuming characters',
						categories: ['All tokens', 'Group constructs',],
					},
					{
						name: 'Negative lookahead',
						example: '(?!...)',
						description: 'Starting at the current position in the expression, ensures that the given pattern will not match. Does not consume characters.',
						categories: ['All tokens', 'Group constructs',],
					},
					{
						name: 'Positive lookbehind',
						example: '(?<=...)',
						description: 'Ensures that the given pattern will match, ending at the current position in the expression. The pattern be of variable width. Does not consume any characters.',
						categories: ['All tokens', 'Group constructs',],
					},
					{
						name: 'Negative lookbehind',
						example: '(?<!...)',
						description: 'Ensures that the given pattern would not match and end at the current position in the expression. The pattern be of variable width. Does not consume characters.',
						categories: ['All tokens', 'Group constructs',],
					},
					{
						name: 'Zero or one quantifier',
						example: 'a?',
						description: 'Matches an `a` character or nothing.',
						categories: ['All tokens', 'Common tokens', 'Quantifiers',],
					},
					{
						name: 'Zero or more quantifier',
						example: 'a*',
						description: 'Matches zero or more consecutive `a` characters.',
						categories: ['All tokens', 'Common tokens', 'Quantifiers',],
					},
					{
						name: 'One or more quantifier',
						example: 'a+',
						description: 'Matches one or more consecutive `a` characters.',
						categories: ['All tokens', 'Common tokens', 'Quantifiers',],
					},
					{
						name: 'Exact number quantifier',
						example: 'a{3}',
						description: 'Matches exactly 3 consecutive `a` characters.',
						categories: ['All tokens', 'Common tokens', 'Quantifiers',],
					},
					{
						name: 'N or more quantifier',
						example: 'a{3,}',
						description: 'Matches at least 3 consecutive `a` characters.',
						categories: ['All tokens', 'Common tokens', 'Quantifiers',],
					},
					{
						name: 'Between N1 and N2 quantifier',
						example: 'a{3,6}',
						description: 'Matches between 3 and 6 (inclusive) consecutive `a` characters.',
						categories: ['All tokens', 'Common tokens', 'Quantifiers',],
					},
					{
						name: 'Lazy quantifier',
						example: 'a*?',
						description: 'Matches as few characters as possible.',
						categories: ['All tokens', 'Quantifiers',],
					},
					{
						name: 'Start of string',
						example: '^',
						description: 'Matches the start of a string without consuming any characters. If multiline mode is used, this will also match immediately after a newline character.',
						categories: ['All tokens', 'Common tokens', 'Anchors',],
					},
					{
						name: 'End of string',
						example: '$',
						description: 'Matches the end of a string without consuming any characters. If multiline mode is used, this will also match immediately before a newline character.',
						categories: ['All tokens', 'Common tokens', 'Anchors',],
					},
					{
						name: 'A word boundary',
						example: '\\b',
						description: 'Matches, without consuming any characters, immediately between a character matched by \\w and a character not matched by \\w (in either order). It cannot be used to separate non words from words.',
						categories: ['All tokens', 'Common tokens', 'Anchors',],
					},
					{
						name: 'Non-word boundary',
						example: '\\B',
						description: 'Matches, without consuming any characters, at the position between two characters matched by \\w.',
						categories: ['All tokens', 'Common tokens', 'Anchors',],
					},
					{
						name: 'Global flag',
						example: 'g',
						description: 'Tells the engine not to stop after the first match has been found, but rather to continue until no more matches can be found.',
						categories: ['All tokens', 'Flags/Modifiers',],
					},
					{
						name: 'Multiline flag',
						example: 'm',
						description: 'The ^ and $ anchors now match at the beginning/end of each line respectively, instead of beginning/end of the entire string.',
						categories: ['All tokens', 'Flags/Modifiers',],
					},
					{
						name: 'Case insensitive flag',
						example: 'i',
						description: 'A case insensitive match is performed, meaning capital letters will be matched by non-capital letters and vice versa.',
						categories: ['All tokens', 'Flags/Modifiers',],
					},
					{
						name: 'Sticky flag',
						example: 'y',
						description: 'The expression will only match from its lastIndex position and ignores the global (g) flag if set. Because each search in RegExr is discrete, this flag has no further impact on the displayed results.',
						categories: ['All tokens', 'Flags/Modifiers',],
					},
					{
						name: 'Unicode support flag',
						example: 'u',
						description: 'Allows regex to match unicode characters via dotall and unicode escape sequences, as well as ES6 unicode code point escapes such as \\u{1D306}',
						categories: ['All tokens', 'Flags/Modifiers',],
					},
					{
						name: 'Dotall flag',
						example: 's',
						description: 'Makes dot (.) match any character, including newline.',
						categories: ['All tokens', 'Flags/Modifiers',],
					},
					{
						name: 'Contents of capture group 1',
						example: '$1',
						description: 'This will return a string with the contents from the first capture group. The number, in this case 1, can be any number as long as it corresponds to a valid capture group.',
						categories: ['All tokens', 'Substitution'],
					},
					{
						name: 'Contents before match',
						example: '$`',
						description: 'This will return a portion of the source string that precedes the match.',
						categories: ['All tokens', 'Substitution'],
					},
					{
						name: 'Contents after match',
						example: '$\'',
						description: 'This will return a portion of the source string that follows the match.',
						categories: ['All tokens', 'Substitution'],
					},
					{
						name: 'Complete match',
						example: '$&',
						description: 'This will return a string with the complete match result from the regex.',
						categories: ['All tokens', 'Substitution'],
					},
					{
						name: 'Escaped $',
						example: '$$',
						description: 'Inserts a dollar sign character ($).',
						categories: ['All tokens', 'Substitution'],
					},
				]
			}
		},

		computed: {
			valueComp: {
				get() {
					return this.value;
				},
				set(value) {
					this.$emit('input', value);
				}
			},
			regexBody: {
				get() {
					let valueRxRes = this.valueComp.match(/^\/([^]*)\/(\w*?)$/);
					if (valueRxRes) {
						return valueRxRes[1];
					} else {
						return '';
					}
				},
				set(value) {
					this.valueComp = `/${value}/${this.flags.join('')}`;
				}
			},
			flags: {
				get() {
					let valueRxRes = this.valueComp.match(/^\/([^]*)\/(\w*?)$/);
					if (valueRxRes) {
						return valueRxRes[2].split('');
					} else {
						return [];
					}
				},
				set(value) {
					this.valueComp = `/${this.regexBody}/${value.join('')}`;
				}
			},
			currentReferenceItems() {
				return this.referenceItems.filter(item => item.categories.indexOf(this.currentCategory) !== -1);
			},
		},

		watch: {
			valueComp() {
				this.error = '';
				this.coloredRx = '';
				if (this.regexBody !== '') {
					try {
						this.processRegExp();
					} catch (e) {
						// console.error(e);
						this.coloredRx = `<span class="error">${_.escape(this.regexBody)}</span>`.replace(/[\n\r\t\v]/g, '');
						this.error = 'Invalid regular expression pattern.';
						this.$refs['expl_collaps'].innerHTML = '';
						this.$refs['str_match_output'].innerHTML = '';
						this.$refs['str_replace_output'].innerHTML = '';
					}
					this.testMatch();
					this.testReplace();
				} else {
					this.$refs['expl_collaps'].innerHTML = '';
					this.$refs['str_match_output'].innerHTML = '';
					this.$refs['str_replace_output'].innerHTML = '';
				}
				this.htmlContent = this.coloredRx.replace(/[\n\r\t\v]/g, '');
			},
			testStrValue() {
				this.testMatch();
				this.testReplace();
			},
			replaceStrValue() {
				this.testReplace();
			}
		},

		methods: {
			onUpdateContent(val) {
				this.$set(this, 'regexBody', val);
			},
			onUpdateFlags(val) {
				this.$set(this, 'flags', val);
			},
			onUpdateFullRx({body, flags}) {
				this.$set(this, 'valueComp', `/${body}/${flags.join('')}`);
			},
			processRegExp() {
				this.rxNamedGroups = [];
				const tree = regexpTree.parser.setOptions({captureLocations: true}).parse(this.valueComp);

				this.$refs['expl_collaps'].innerHTML = this.getVariant(tree).fn(tree);
			},
			testMatch() {
				if (this.testStrValue !== '') {
					try {
						let global = this.flags.indexOf('g') !== -1;
						let rx = RegExp(this.regexBody, this.flags.join(''));
						let matches = [];
						let match;
						do {
							match = rx.exec(this.testStrValue);
							if (match) {
								if (match.index === rx.lastIndex) rx.lastIndex++;

								let groups = match.map((group, i) => {
									let namedGroup = this.rxNamedGroups.find(namedGroup => namedGroup.number === i);
									return {
										name: namedGroup ? namedGroup.name : i,
										text: group
									}
								});
								groups.splice(0, 1);

								matches.push({
									full: match[0],
									begin: match.index,
									end: match.index + match[0].length,
									groups
								});
							}
						} while (match && global);

						let testResult = matches.map((match, i) => `<table class="match_container"><tr><th colspan="2" class="match_title">Match ${i + 1}</th></tr>
<tr class="match_row"><td class="match_label">Full match (${match.begin}-${match.end}):</td><td class="match_text">${_.escape(match.full)}</td></tr>
${match.groups.reduce((res, group) => res + `<tr class="match_row"><td class="match_label">Group \`${group.name}\`:</td><td class="match_text">${_.escape(group.text)}</td></tr>`, '')}
</table>`);

						this.$refs['str_match_output'].innerHTML = testResult.join('');
					} catch (e) {
						this.$refs['str_match_output'].innerHTML = '';
					}
				}
			},
			testReplace() {
				this.replaceStringError = '';
				let replaceStrValue = '';

				try {
					replaceStrValue = unraw(this.replaceStrValue, true);
				} catch (e) {
					this.replaceStringError = 'Malformed replacement string';
					this.$refs['str_replace_output'].innerHTML = '';
					return;
				}

				try {
					let rx = RegExp(this.regexBody, this.flags.join(''));

					let replaceRes = this.testStrValue.replace(rx, replaceStrValue);

					this.$refs['str_replace_output'].innerHTML = _.escape(replaceRes);
				} catch (e) {
					this.$refs['str_replace_output'].innerHTML = '';
				}
			},
			getVariant(node) {
				return this.types[node.type].variants.find(variant => variant.condition(node));
			},
			convertBackrefToSymb(node) {
				let value = `\\${node.number}`;
				let kind;
				let symbol;
				let symbolDecimal = String.fromCharCode(node.number);
				let symbolOct = String.fromCharCode(parseInt(`${node.number}`, 8));
				let symbolSimple = `${node.number}`;
				let escaped = false;
				if (RegExp(node.value).test(symbolDecimal)) {
					kind = 'decimal';
					symbol = symbolDecimal;
				} else if (RegExp(node.value).test(symbolOct)) {
					kind = 'oct';
					symbol = symbolOct;
				} else if (RegExp(node.value).test(symbolSimple)) {
					kind = 'simple';
					symbol = symbolSimple;
					escaped = true;
				}

				return {
					"type": "Char",
					"value": value,
					"kind": kind,
					"symbol": symbol,
					"escaped": escaped,
					"loc": {
						"source": value,
					}
				}
			},
			selectReferenceCategory(category) {
				this.currentCategory = category;
			},
			resetReferenceCategory() {
				this.currentCategory = '';
				this.currentReference = '';
				this.currentDescriptionHeight = 0;
			},
			toggleRefDescription(refItem) {
				if (this.currentReference === refItem.name) {
					this.currentReference = false;
					this.currentDescriptionHeight = 0;
				} else {
					this.currentReference = refItem.name;
					this.currentDescriptionHeight = this.$refs[`reference_item_${refItem.name}`][0].clientHeight;
				}
			},
		},

		mounted() {
			if (this.regexBody !== '') {
				try {
					this.processRegExp();
				} catch (e) {
					// console.error(e);
					this.coloredRx = `<span class="error">${_.escape(this.regexBody)}</span>`.replace(/[\n\r\t\v]/g, '');
					this.error = 'Invalid regular expression pattern.';
				}
				this.testMatch();
				this.testReplace();
				this.htmlContent = this.coloredRx.replace(/[\n\r\t\v]/g, '');
			}
		},

		components: {
			CustomRxInput
		},
	};
</script>