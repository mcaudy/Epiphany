<template lang="pug">
	.noteBar
		nav: ul(:class="{'transparent' : !isPreview && !isToolbarEnabled }")
			li(v-if="isNoteSelected && !isOutlineSelected", :class="{ 'entry-hidden': isPreview || !isToolbarEnabled }")
				a(@click="menu_image", href="#")
					i.coon-image
					|  Image
			li(v-if="isNoteSelected && !isOutlineSelected", :class="{ 'entry-hidden': isPreview || !isToolbarEnabled }")
				div: dropdown(:visible="table_visible", :position="position_left", v-on:clickout="table_visible = false")
					span.link(@click="table_visible = !table_visible")
						i.coon-table
						|  Table
					.dialog(slot="dropdown")
						.table-dialog(@click="close_table")
							table.select-table-size(cellpadding="2", @mouseleave="tableClean")
								tr(v-for="r in table_max_row")
									td(v-for="c in table_max_column", @click="tableSelect(r,c)", @mouseenter="tableHover(r,c)", ref="tablesizetd")
							span(v-if="table_hover_row > 0") {{ table_hover_row }} x {{ table_hover_column }}
							span(v-else)
								| Select the Table Size
			li(v-if="isNoteSelected && !isOutlineSelected", :class="{ 'entry-hidden': isPreview || !isToolbarEnabled }")
				a(@click="menu_checkMark", href="#")
					i.coon-check-square
					|  Checkbox
			li(v-if="isNoteSelected && !isOutlineSelected", :class="{ 'entry-hidden': isPreview || !isToolbarEnabled }")
				a(@click="menu_codeBlock", href="#")
					i.coon-code
					|  Code block

			li(v-if="isNoteSelected && !isOutlineSelected", :class="{ 'entry-hidden': !isPreview || !noteHeadings || noteHeadings.length < 2 }")
				div: dropdown(:visible="headings_visible", :position="position_left", v-on:clickout="headings_visible = false")
					span.link(@click="headings_visible = !headings_visible")
						i.coon-list
						|  Headers
					.dialog(slot="dropdown")
						.headings-dialog(@click="close_headings")
							a.h(v-for="head in noteHeadings", @click.prevent="jumpTo(head.id)", :class="'hlvl'+head.level", v-html="head.text")

			li.right-align(:class="{ 'entry-hidden': isPreview || !isToolbarEnabled }")
				div: dropdown(:visible="fontstyle_visible", :position="position_right", v-on:clickout="fontstyle_visible = false")
					span.link(@click="fontstyle_visible = !fontstyle_visible")
						i.coon-size
						|  Text Style
					.dialog(slot="dropdown"): ul.fontsize-dialog
						li: a(@click.prevent="menu_fontstyle('normal')", href="#")
							i.coon-check-circle(v-if="!useMonospace")
							i.coon-circle.faded(v-else)
							|  Normal
						li: a(@click.prevent="menu_fontstyle('monospace')", href="#")
							i.coon-check-circle(v-if="useMonospace")
							i.coon-circle.faded(v-else)
							|  Monospace

			li.right-align(:class="{ 'entry-hidden': !isToolbarEnabled }")
				div: dropdown(:visible="fontsize_visible", :position="position_right", v-on:clickout="fontsize_visible = false")
					span.link(@click="fontsize_visible = !fontsize_visible")
						i.coon-size
						|  Text Size
					.dialog(slot="dropdown"): ul.fontsize-dialog
						li: a(@click.prevent="menu_fontsize(10)", href="#")
							i.coon-check-circle(v-if="fontsize == 10")
							i.coon-circle.faded(v-else)
							|  10
						li: a(@click.prevent="menu_fontsize(12)", href="#")
							i.coon-check-circle(v-if="fontsize == 12")
							i.coon-circle.faded(v-else)
							|  12
						li: a(@click.prevent="menu_fontsize(14)", href="#")
							i.coon-check-circle(v-if="fontsize == 14")
							i.coon-circle.faded(v-else)
							|  14
						li: a(@click.prevent="menu_fontsize(15)", href="#")
							i.coon-check-circle(v-if="fontsize == 15")
							i.coon-circle.faded(v-else)
							|  15
						li: a(@click.prevent="menu_fontsize(16)", href="#")
							i.coon-check-circle(v-if="fontsize == 16")
							i.coon-circle.faded(v-else)
							|  16
						li: a(@click.prevent="menu_fontsize(18)", href="#")
							i.coon-check-circle(v-if="fontsize == 18")
							i.coon-circle.faded(v-else)
							|  18
						li: a(@click.prevent="menu_fontsize(20)", href="#")
							i.coon-check-circle(v-if="fontsize == 20")
							i.coon-circle.faded(v-else)
							|  20
						li: a(@click.prevent="menu_fontsize(24)", href="#")
							i.coon-check-circle(v-if="fontsize == 24")
							i.coon-circle.faded(v-else)
							|  24
			
			li.right-align(v-if="isNoteSelected && !isOutlineSelected", :class="{ 'entry-hidden': !isToolbarEnabled }")
				div: dropdown(:visible="properties_visible", :position="position_right", v-on:clickout="properties_visible = false")
					span.link(@click="properties_visible = !properties_visible")
						i.coon-info
						|  Properties
					.dialog(slot="dropdown")
						.properties-dialog(@click="close_properties")
							table
								tr
									td: strong Path: 
									td.right: span(style="white-space: pre;") {{ note.relativePathNoFileName }}
							hr
							table
								tr
									td: strong Line Count: 
									td.right: span {{ note.properties.lineCount }}
								tr
									td: strong Word Count: 
									td.right: span {{ note.properties.wordCount }}
								tr
									td: strong Char Count: 
									td.right: span {{ note.properties.charCount }}
							hr
							table
								tr
									td: strong Modified: 
									td.right: span {{ note.updatedAt.format('MMM DD, YYYY') }}
								tr
									td: strong Created: 
									td.right: span {{ note.createdAt.format('MMM DD, YYYY') }}
							hr
							form.new-metadata-form(@submit="newMetadata")
								table(@click.prevent.stop="")
									tr(
										v-for="metakey in note.metadataKeys"
										v-if="metakey != 'createdAt' && metakey != 'updatedAt' && metakey != 'starred' && note.metadata[metakey]"
									)
										td: strong {{ metakey }}
										td.right: span {{ note.metadata[metakey] }}
									tr
										td: strong
											select(name="metakey", required, ref="keyinput")
												option(value="") ---
												option Author
												option Copyright
												option Language
												option Subtitle
												option Title
												option Web
										td.right: span
											input(type="text", name="metavalue", ref="valueinput")

			li.right-align(v-if="isPreview && isNoteSelected && !isOutlineSelected")
				a(@click="openShare", href="#", title="Share")
					i.coon-share-2
					|  Share

			li.right-align(v-if="isNoteSelected && !isOutlineSelected")
				a(@click="togglePreview", href="#", title="Preview")
					template(v-if="isPreview")
						i.coon-eye-off
						|  Hide Preview
					template(v-else)
						i.coon-eye
						|  Show Preview
</template>

<script>
	import { remote } from "electron";
	import myDropdown from 'vue-my-dropdown';

	export default {
		name: 'noteMenu',
		props: {
			'note'             : Object,
			'isFullScreen'     : Boolean,
			'isPreview'        : Boolean,
			'isToolbarEnabled' : Boolean,
			'isNoteSelected'   : Boolean,
			'isOutlineSelected': Boolean,
			"useMonospace"     : Boolean,
			'fontsize'         : Number,
			'togglePreview'    : Function,
			'sendFlashMessage' : Function
		},
		data() {
			return {
				'fontsize_visible': false,
				'fontstyle_visible': false,
				'properties_visible': false,
				'headings_visible': false,
				'table_visible': false,
				'table_max_row': 10,
				'table_max_column': 10,
				'table_hover_row': 0,
				'table_hover_column': 0,
				'position_left': [ "left", "top", "left", "top" ],
				'position_right': [ "right", "top", "right", "top" ]
			};
		},
		computed: {
			noteHeadings() {

				function getNodeText(oDiv) {
					var firstText = "";
					var whitespace = /^\s*$/;
					for (var i = 0; i < oDiv.childNodes.length; i++) {
						var curNode = oDiv.childNodes[i];
						if (curNode.nodeType === Node.TEXT_NODE && !(whitespace.test(curNode.nodeValue))) {
							firstText = curNode.nodeValue;
							break;
						}
					}
					return firstText;
				}

				if (!this.isPreview) return [];
				var note_heading = [];
				var headings = document.querySelectorAll(".my-editor-preview .epiphany-heading");
				for (var i=0; i<headings.length; i++) {
					var node = headings[i];
					note_heading.push({
						id   : node.getAttribute("name"),
						text : getNodeText(node.parentNode),
						level: parseInt(node.parentNode.tagName.replace(/h/i,""))
					});
				}
				return note_heading;
			}
		},
		components: {
			'dropdown': myDropdown
		},
		methods: {
			codeMirror() {
				return this.$root.codeMirror;
			},
			close_properties() {
				this.properties_visible = false;
			},
			close_table() {
				this.table_visible = false;
				this.tableClean();
			},
			close_headings() {
				this.headings_visible = false;
			},
			tableClean() {
				for (var i=0;i<this.table_max_row;i++) {
					for (var j=0;j<this.table_max_column;j++) {
						this.$refs.tablesizetd[i*this.table_max_row+j].classList.remove("selected");
					}
				}
				this.table_hover_row = 0;
				this.table_hover_column = 0;
			},
			tableSelect(row, column) {
				this.table_visible = false;

				var markdown_table = []
				for (var i=0;i<row;i++) {
					var column_table = [];
					for (var j=0;j<column;j++) {
						column_table.push('....');
					}
					markdown_table.push(column_table);
				}

				var table = require('markdown-table');
				var cm = this.codeMirror();
				var cursor = cm.getCursor();

				if(cursor.ch == 0){
					if(cm.doc.getLine(cursor.line).length > 0){
						cm.doc.replaceRange( table(markdown_table)+'\n', cursor);
					} else {
						cm.doc.replaceRange( table(markdown_table), cursor);
					}
				} else {
					cursor.ch = cm.doc.getLine(cursor.line).length;
					cm.doc.replaceRange( '\n'+table(markdown_table), cursor);
					cursor.line += 1;
				}
				
				cm.doc.setCursor({
					line: cursor.line,
					ch: 0
				});
				cm.focus();
			},
			tableHover(row, column) {
				this.tableClean();
				for (var i=0;i<row;i++) {
					for (var j=0;j<column;j++) {
						this.$refs.tablesizetd[i*this.table_max_row+j].classList.add('selected');
					}
				}
				this.table_hover_row = row;
				this.table_hover_column = column;
			},
			openShare() {
				this.$root.open_share_url();
			},
			menu_fontsize(size) {
				this.$parent.fontsize = size;
				this.fontsize_visible = false;
			},
			menu_fontstyle(style) {
				switch(style) {
					case "normal":
						this.$parent.useMonospace = false;
						break;
					case "monospace":
						this.$parent.useMonospace = true;
						break;
				}
				this.fontstyle_visible = false;
			},
			menu_image() {
				if (this.table_visible) {
					this.close_table();
					return;
				}

				var cm = this.codeMirror();
				var cursor = cm.getCursor();

				if(cursor.ch == 0){
					if(cm.doc.getLine(cursor.line).length > 0){
						cm.doc.replaceRange( "![]()\n", cursor);
					} else {
						cm.doc.replaceRange( "![]()", cursor);
					}
				} else {
					cursor.ch = cm.doc.getLine(cursor.line).length;
					cm.doc.replaceRange( "\n![]()", cursor);
					cursor.line += 1;
				}
				
				cm.doc.setCursor({
					line: cursor.line,
					ch: 4
				});
				cm.focus();
			},
			menu_codeBlock() {
				if (this.table_visible) {
					this.close_table();
					return;
				}

				var cm = this.codeMirror();
				var cursor = cm.getCursor();

				if(cursor.ch == 0){
					if(cm.doc.getLine(cursor.line).length > 0){
						cm.doc.replaceRange( "```\n\n```\n", cursor);
					} else {
						cm.doc.replaceRange( "```\n\n```", cursor);
					}
				} else {
					cursor.ch = cm.doc.getLine(cursor.line).length;
					cm.doc.replaceRange( "\n```\n\n```", cursor);
					cursor.line += 1;
				}
				
				
				cm.doc.setCursor({
					line: cursor.line+1,
					ch: 0
				});
				cm.focus();
			},
			menu_checkMark() {
				if (this.table_visible) {
					this.close_table();
					return;
				}

				var cm = this.codeMirror();
				var cursor = cm.getCursor();
				
				if(cursor.ch == 0){
					if(cm.doc.getLine(cursor.line).length > 0){
						cm.doc.replaceRange( "* [ ] \n", cursor);
					} else {
						cm.doc.replaceRange( "* [ ] ", cursor);
					}
				} else {
					cursor.ch = cm.doc.getLine(cursor.line).length;
					cm.doc.replaceRange( "\n* [ ] ", cursor);
					cursor.line += 1;
				}
				
				cm.doc.setCursor({
					line: cursor.line,
					ch: cursor.ch+5
				});
				cm.focus();
			},
			newMetadata(e) {
				e.preventDefault();
				this.properties_visible = false;
				this.note.setMetadata(this.$refs.keyinput.value, this.$refs.valueinput.value);
				this.note.saveModel();
				this.sendFlashMessage(2000, 'info', 'New metadata added');
				this.$refs.valueinput.value = '';
				this.$refs.keyinput.value = '';
				this.properties_visible = true;
			},
			jumpTo(anchor) {
				var editor = document.querySelector('.my-editor-preview');
				var pos = document.querySelector('.my-editor-preview #'+anchor);
				editor.scrollTop = Math.max(0, pos.offsetTop - pos.clientHeight - 10);
			}
		}
	}
</script>