<template lang="pug">
	.codemirror-div
</template>

<script>
	import fs from "fs";
	import path from "path";

	import _ from "lodash";

	const Image = require('../models').Image;

	import Vue from "vue";

	import electron from "electron";
	const { remote, shell, clipboard } = electron;
	const { Menu, MenuItem } = remote;

	import { copyText, cutText, pasteText, selectAllText } from "../codemirror/elutils";

	const IMAGE_TAG_TEMP = _.template('![<%- filename %>](<%- fileurl %>)\n');

	require('codemirror/lib/codemirror.css');
	require('codemirror/addon/search/matchesonscrollbar.css');

	import CodeMirror from "codemirror";

	require('codemirror/addon/search/searchcursor');
	require('codemirror/addon/search/matchesonscrollbar');
	require('../codemirror/piledsearch');
	require('codemirror/addon/edit/closebrackets');
	require('codemirror/addon/mode/overlay');
	require('../codemirror/placeholder');
	require('codemirror/mode/xml/xml');
	require('codemirror/mode/markdown/markdown');
	require('codemirror/mode/gfm/gfm');
	require('codemirror/mode/rst/rst');
	require('../codemirror/piledmd');
	require('codemirror/mode/python/python');
	require('codemirror/mode/javascript/javascript');
	require('codemirror/mode/coffeescript/coffeescript');
	require('codemirror/mode/pug/pug');
	require('codemirror/mode/css/css');
	require('codemirror/mode/htmlmixed/htmlmixed');
	require('codemirror/mode/clike/clike');
	require('codemirror/mode/http/http');
	require('codemirror/mode/ruby/ruby');
	require('codemirror/mode/lua/lua');
	require('codemirror/mode/go/go');
	require('codemirror/mode/php/php');
	require('codemirror/mode/perl/perl');
	require('codemirror/mode/swift/swift');
	require('codemirror/mode/go/go');
	require('codemirror/mode/sql/sql');
	require('codemirror/mode/yaml/yaml');
	require('codemirror/mode/shell/shell');
	require('codemirror/mode/commonlisp/commonlisp');
	require('codemirror/mode/clojure/clojure');
	require('codemirror/mode/meta');
	require('../codemirror/piledmap');

	function countWords(text) {
		text = text.replace(/\[[\w\s]+?\]/g, " ").replace(/[^\w\-’'"_]/g, " ").replace(/[\s\r\n]+/g, " ").replace(/\s\W\s/g, " ");
		var words = text.split(" ").filter((str) => {
			return str.length > 0
		});
		return words.length;
	}

	function countLineBreaks(text) {
		var lines = text.split("\n");
		return lines.length;
	}

	export default {
		name: "codemirror",
		props: {
			"note": Object,
			"isFullScreen": Boolean,
			"isPreview": Boolean,
			"useMonospace": Boolean,
			"togglePreview": Function,
			"search": String
		},
		data() {
			return {
				cm: null
			};
		},
		mounted() {
			this.newCodemirrorInstance();
		},
		methods: {
			newCodemirrorInstance() {
				if (this.$root.$refs.refNoteFooter) {
					this.$root.$refs.refNoteFooter.row = 0;
					this.$root.$refs.refNoteFooter.column = 0;
					this.$root.$refs.refNoteFooter.selection = 0;
				}

				var cm = CodeMirror(this.$el, {
					mode             : "piledmd",
					lineNumbers      : false,
					lineWrapping     : true,
					theme            : "default",
					keyMap           : "piledmap",
					extraKeys        : {
						'Ctrl-Y'      : () => { pasteText(cm, this.note); },
						'Ctrl-V'      : () => { pasteText(cm, this.note); },
						'Alt-V'       : () => { pasteText(cm, this.note); },
						'Shift-Ctrl-V': () => { this.togglePreview(); },
						'Alt-P'       : () => { this.togglePreview(); }
					},
					indentUnit        : 4,
					smartIndent       : true,
					tabSize           : 4,
					indentWithTabs    : true,
					cursorBlinkRate   : 540,
					addModeClass      : true,
					autoCloseBrackets : true,
					scrollbarStyle    : 'native',
					cursorScrollMargin: 10,
					placeholder       : '',
					value             : this.note ? new CodeMirror.Doc(this.note.body, 'piledmd') : undefined
				});
				this.cm = cm;
				this.$root.codeMirror = cm;

				cm.on('change', this.updateNoteBody);
				cm.on('blur', this.updateNoteBeforeSaving);

				cm.on('drop', (cm, event) => {
					if (event.dataTransfer.files.length > 0) {
						var p = cm.coordsChar({ top: event.y, left: event.x });
						cm.setCursor(p);
						this.uploadFiles(cm, event.dataTransfer.files);
					} else {
						return true;
					}
				});

				var isLinkState = (type) => {
					if (!type) {
						return false }
					var types = type.split(' ');
					return (_.includes(types, 'link') ||
						_.includes(types, 'piled-link-href') ||
						_.includes(types, 'link')) && !_.includes(types, 'piled-formatting');
				};
				cm.on('contextmenu', (cm, event) => {
					// Makidng timeout Cause codemirror's contextmenu handler using setTimeout on 50ms or so.
					setTimeout(() => {
						var menu = new Menu();

						menu.append(new MenuItem({
							label      : 'Cut',
							accelerator: 'CmdOrCtrl+X',
							click      : () => { cutText(cm) }
						}));

						menu.append(new MenuItem({
							label      : 'Copy',
							accelerator: 'CmdOrCtrl+C',
							click      : () => { copyText(cm) }
						}));

						menu.append(new MenuItem({
							label      : 'Paste',
							accelerator: 'CmdOrCtrl+V',
							click      : () => { pasteText(cm, this.note) }
						}));

						menu.append(new MenuItem({ type: 'separator' }));
						menu.append(new MenuItem({
							label      : 'Select All',
							accelerator: 'CmdOrCtrl+A',
							click      : () => { selectAllText(cm) }
						}));

						menu.append(new MenuItem({ type: 'separator' }));
						menu.append(new MenuItem({
							label      : 'Attach Image',
							accelerator: 'Shift+CmdOrCtrl+A',
							click      : () => { this.uploadFile() }
						}));

						var c = cm.getCursor();
						var token = cm.getTokenAt(c, true);
						if (isLinkState(token.type)) {
							var s = cm.getRange({ line: c.line, ch: token.start < 2 ? token.start : token.start-2 }, { line: c.line, ch: token.state.overlayPos || token.end });
							s = s.replace(/^\W+/, '');
							s = s.replace(/^s:\/\//, 'https://');
							s = s.replace(/^p:\/\//, 'http://');
							s = s.replace(/\)$/, '');

							menu.append(new MenuItem({ type: 'separator' }));
							menu.append(new MenuItem({
								label: 'Copy Link',
								click: () => { clipboard.writeText(s) }
							}));
							menu.append(new MenuItem({
								label: 'Open Link In Browser',
								click: () => {
									shell.openExternal(s)
								}
							}));
						} else {
							/*menu.append(new MenuItem({
								label  : 'Copy Link',
								enabled: false
							}));
							menu.append(new MenuItem({
								label  : 'Open Link In Browser',
								enabled: false
							}));*/
						}
						menu.append(new MenuItem({ type: 'separator' }));
						menu.append(new MenuItem({ label: 'Toggle Preview', click: () => { this.togglePreview() } }));
						menu.popup(remote.getCurrentWindow());
					}, 90);
				});
				cm.on('cursorActivity', (cm, event) => {
					var sel = cm.getSelection();
					var c = cm.getCursor();
					if (this.$root.$refs.refNoteFooter) {
						this.$root.$refs.refNoteFooter.row = c.line;
						this.$root.$refs.refNoteFooter.column = c.ch;
						this.$root.$refs.refNoteFooter.selection = sel.length;
						if (sel.length == 0) {
							this.$root.$refs.refNoteFooter.wordscount = countWords(cm.getValue());
							this.$root.$refs.refNoteFooter.linebreaks = 0;
						} else {
							this.$root.$refs.refNoteFooter.wordscount = countWords(sel);
							this.$root.$refs.refNoteFooter.linebreaks = countLineBreaks(sel);
						}
					}
				});

				cm.on('swapDoc', (cm, old_doc) => {
					if (this.$root.$refs.refNoteFooter) {
						this.$root.$refs.refNoteFooter.wordscount = countWords(cm.getValue());
					}
				});

				if (this.useMonospace) {
					this.$el.classList.add("codemirror-monospace");
				}

				this.initFooter();
				this.runSearch();
			},
			initFooter() {
				if (this.$root.$refs.refNoteFooter) {
					this.$root.$refs.refNoteFooter.row = 0;
					this.$root.$refs.refNoteFooter.column = 0;
					this.$root.$refs.refNoteFooter.selection = 0;

					this.$root.$refs.refNoteFooter.wordscount = countWords(this.cm.getValue());
				}
			},
			uploadFile() {
				var notePaths = remote.dialog.showOpenDialog({
					title: 'Attach Image',
					filters: [{
						name      : 'Markdown',
						extensions: [
							'png', 'jpeg', 'jpg', 'bmp',
							'gif', 'tif', 'ico'
						]
					}],
					properties: ['openFile', 'multiSelections']
				});
				if (!notePaths || notePaths.length == 0) {
					return }

				var files = notePaths.map((notePath) => {
					var name = path.basename(notePath);
					return { name: name, path: notePath }
				});
				this.uploadFiles(this.cm, files);
			},
			uploadFiles(cm, files) {
				files = Array.prototype.slice.call(files, 0, 5);
				_.forEach(files, (f) => {
					try {
						var image = Image.fromBinary(f.name, f.path, this.note);
					} catch (e) {
						this.$message('error', 'Failed to load and save image', 5000);
						console.log(e);
						return
					}
					cm.doc.replaceRange(
						IMAGE_TAG_TEMP({ filename: f.name, fileurl: image.coonURL }),
						cm.doc.getCursor()
					);
					this.$message('info', 'Saved image to ' + image.makeFilePath());
				});
			},
			updateNoteBody: _.debounce(function () {
					this.note.body = this.cm.getValue();
				}, 1000, {
					leading : true,
					trailing: true,
					maxWait : 5000
				}
			),
			updateNoteBeforeSaving() {
				this.note.body = this.cm.getValue();
			},
			runSearch() {
				if (this.note && this.search && this.search.length > 1) {
					this.$nextTick(() => {
						CodeMirror.commands.setSearch(this.cm, this.search.toLowerCase());
					});
				} else {
					CodeMirror.commands.undoSearch(this.cm);
				}
			},
			refreshCM() {
				this.cm.refresh();
			},
			refreshNoteBody() {
				var doc;
				if (this.note.body) {
					doc = new CodeMirror.Doc(this.note.body, 'piledmd');
				}
				this.note.doc = doc;
				if (doc) {
					if(doc.cm) doc.cm = null;
					this.cm.swapDoc(doc);
				}
			}
		},
		watch: {
			isFullScreen() {
				if (this.cm) {
					this.$nextTick(() => {
						setTimeout(() => {
							this.refreshCM();
							this.cm.focus();
						}, 300);
					});
				}
			},
			isPreview() {
				if (!this.isPreview) {
					this.$nextTick(() => {
						this.refreshCM();
						this.cm.focus();
					});
				}
			},
			note(value) {
				if (!value || !value.title || !this.cm) return;

				this.cm.scrollTo(0,0);
				this.cm.getDoc().setValue(value.body);
				this.initFooter();
				this.runSearch();
			},
			search() {
				this.runSearch();
			},
			useMonospace() {
				if (this.useMonospace) {
					this.$el.classList.add("codemirror-monospace");
				} else {
					this.$el.classList.remove("codemirror-monospace");
				}
			}
		}
	}
</script>