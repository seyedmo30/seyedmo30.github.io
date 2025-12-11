چیزایی که باید رو vscode نصب شه :

Git Graph  ----    mhutchie.git-graph

git lens  -  eamodio.gitlens

Error Lens  ---    usernamehw.errorlens

Paste JSON as Code ---  quicktype.quicktype

Prettier - Code formatter ---  esbenp.prettier-vscode

bookmarks - برای علامت زدن روی کدی که مهمه در یه فایل بزرگ

streetsidesoftware.code-spell-checker  غلط املایی

AMiner.codegeex  - اتو کامپلیت چینی

 RobertOstermann.inline-parameters-extended     -   نام هر پارام توی فانکشن ها کمرنگ می نویسه

finntenzor.change-case   -   برای تغییر pascal case to camel case  باید ابتدا پکیج زیر رو نصب کنید


## keymap shortcuts

+ go to definition (ctrl + shit + click)

میره جایی که تابع تعریف شده

+ go to impelemention  (ctrl + shift + space)

اگر اینترفیس باشه جایی که اون رو کانکریت کرده

+ go to refrence  (ctrl + click)

میره جاهایی که از این فانکشن استفاده شده

+ go to symbol

نمایش فانکشن ها ، استراکت ها و کلاس ها در صفحه

+ peek definition (ctrl + space)

برای دسترسی به پارامتر های تابع

+ go back (alt + leftArrow)

کار قبلی که انجام دادیم

+ go forward (alt + rightArrow)

بازگشت به کاری که انجام دادیم

+ fold /unfold all 

 کولپس  و اکسپند تمام صفحه

## debug

برای debug توی main.go 
#### ...... project/.vscode/launch.json 


#### key map

`~/.config/Code/User/keybindings.json`

```

// Place your key bindings in this file to override the defaultsauto[]
[
    {
        "key": "alt+left",
        "command": "-workbench.action.terminal.focusPreviousPane",
        "when": "terminalFocus && terminalHasBeenCreated || terminalFocus && terminalProcessSupported"
    },
    {
        "key": "alt+left",
        "command": "-gitlens.key.alt+left",
        "when": "gitlens:key:alt+left"
    },
    {
        "key": "alt+left",
        "command": "workbench.action.navigateBack",
        "when": "canNavigateBack"
    },
    {
        "key": "ctrl+alt+-",
        "command": "-workbench.action.navigateBack",
        "when": "canNavigateBack"
    },
    {
        "key": "alt+right",
        "command": "-workbench.action.terminal.focusNextPane",
        "when": "terminalFocus && terminalHasBeenCreated || terminalFocus && terminalProcessSupported"
    },
    {
        "key": "alt+right",
        "command": "-gitlens.key.alt+right",
        "when": "gitlens:key:alt+right"
    },
    {
        "key": "alt+right",
        "command": "workbench.action.navigateForward",
        "when": "canNavigateForward"
    },
    {
        "key": "ctrl+shift+-",
        "command": "-workbench.action.navigateForward",
        "when": "canNavigateForward"
    },
    {
        "key": "ctrl+shift+space",
        "command": "editor.action.peekDefinition",
        "when": "editorHasDefinitionProvider && editorTextFocus && !inReferenceSearchEditor && !isInEmbeddedEditor"
    },
    {
        "key": "ctrl+shift+f10",
        "command": "-editor.action.peekDefinition",
        "when": "editorHasDefinitionProvider && editorTextFocus && !inReferenceSearchEditor && !isInEmbeddedEditor"
    },
    {
        "key": "ctrl+.",
        "command": "-notebook.cell.openFailureActions",
        "when": "notebookCellFocused && notebookCellHasErrorDiagnostics && !notebookCellEditorFocused"
    },
    {
        "key": "ctrl+.",
        "command": "-editor.action.quickFix",
        "when": "editorHasCodeActionsProvider && textInputFocus && !editorReadonly"
    },
    {
        "key": "ctrl+.",
        "command": "-workbench.action.terminal.showQuickFixes",
        "when": "terminalFocus"
    },
    {
        "key": "ctrl+.",
        "command": "-editor.changeDropType",
        "when": "dropWidgetVisible"
    },
    {
        "key": "ctrl+.",
        "command": "-editor.changePasteType",
        "when": "pasteWidgetVisible"
    },
    {
        "key": "ctrl+.",
        "command": "-acceptSelectedCodeAction",
        "when": "codeActionMenuVisible"
    },
    {
        "key": "ctrl+.",
        "command": "-problems.action.showQuickFixes",
        "when": "problemFocus"
    },
    {
        "key": "ctrl+,",
        "command": "-workbench.action.openSettings"
    },
    {
        "key": "ctrl+,",
        "command": "workbench.action.focusSideBar"
    },
    {
        "key": "ctrl+0",
        "command": "-workbench.action.focusSideBar"
    },
    {
        "key": "ctrl+.",
        "command": "workbench.action.lastEditorInGroup"
    },
    {
        "key": "ctrl+9",
        "command": "-workbench.action.lastEditorInGroup"
    },
    {
        "key": "ctrl+l",
        "command": "-workbench.action.chat.newChat",
        "when": "chatIsEnabled && inChat"
    },
    {
        "key": "ctrl+l",
        "command": "-expandLineSelection",
        "when": "textInputFocus"
    },
    {
        "key": "ctrl+l",
        "command": "-notebook.centerActiveCell",
        "when": "notebookEditorFocused"
    },
    {
        "key": "ctrl+k ctrl+c",
        "command": "-editor.action.addCommentLine",
        "when": "editorTextFocus && !editorReadonly"
    },
    {
        "key": "ctrl+k m",
        "command": "-workbench.action.editor.changeLanguageMode",
        "when": "!notebookEditorFocused"
    },
    {
        "key": "ctrl+k ctrl+alt+c",
        "command": "-workbench.action.addComment"
    },
    {
        "key": "ctrl+k ctrl+alt+down",
        "command": "-editor.action.nextCommentingRange",
        "when": "accessibilityModeEnabled && commentFocused || accessibilityModeEnabled && editorFocus || accessibilityHelpIsShown && accessibilityModeEnabled && accessibleViewCurrentProviderId == 'comments'"
    },
    {
        "key": "ctrl+k ctrl+alt+up",
        "command": "-editor.action.previousCommentingRange",
        "when": "accessibilityModeEnabled && commentFocused || accessibilityModeEnabled && editorFocus || accessibilityHelpIsShown && accessibilityModeEnabled && accessibleViewCurrentProviderId == 'comments'"
    },
    {
        "key": "ctrl+k ctrl+,",
        "command": "-editor.createFoldingRangeFromSelection",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+i",
        "command": "-editor.debug.action.showDebugHover",
        "when": "editorTextFocus && inDebugMode"
    },
    {
        "key": "ctrl+k ctrl+k",
        "command": "-editor.action.defineKeybinding",
        "when": "resource == 'vscode-userdata:/home/ssd/.config/Code/User/keybindings.json'"
    },
    {
        "key": "ctrl+k e",
        "command": "-workbench.files.action.focusOpenEditorsView",
        "when": "workbench.explorer.openEditorsView.active"
    },
    {
        "key": "ctrl+k c",
        "command": "-workbench.files.action.compareWithClipboard"
    },
    {
        "key": "ctrl+k s",
        "command": "-workbench.action.files.saveWithoutFormatting"
    },
    {
        "key": "ctrl+k ctrl+-",
        "command": "-editor.foldAllExcept",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+shift+\\",
        "command": "-workbench.action.splitEditorInGroup",
        "when": "activeEditorCanSplitInGroup"
    },
    {
        "key": "ctrl+k ctrl+\\",
        "command": "-workbench.action.splitEditorLeft"
    },
    {
        "key": "ctrl+k ctrl+\\",
        "command": "-workbench.action.splitEditorOrthogonal"
    },
    {
        "key": "ctrl+k ctrl+\\",
        "command": "-workbench.action.splitEditorRight"
    },
    {
        "key": "ctrl+k ctrl+\\",
        "command": "-workbench.action.splitEditorUp"
    },
    {
        "key": "ctrl+k ctrl+m",
        "command": "-workbench.action.toggleMaximizeEditorGroup",
        "when": "editorPartMaximizedEditorGroup || editorPartMultipleEditorGroups"
    },
    {
        "key": "ctrl+k ctrl+h",
        "command": "-workbench.action.output.toggleOutput",
        "when": "workbench.panel.output.active"
    },
    {
        "key": "ctrl+k z",
        "command": "-workbench.action.toggleZenMode",
        "when": "!isAuxiliaryWindowFocusedContext"
    },
    {
        "key": "ctrl+k shift+enter",
        "command": "-workbench.action.unpinEditor",
        "when": "activeEditorIsPinned"
    },
    {
        "key": "ctrl+k f",
        "command": "-workbench.action.closeFolder",
        "when": "emptyWorkspaceSupport && workbenchState != 'empty'"
    },
    {
        "key": "ctrl+k ctrl+a",
        "command": "-keybindings.editor.addKeybinding",
        "when": "inKeybindings && keybindingFocus"
    },
    {
        "key": "ctrl+k ctrl+e",
        "command": "-keybindings.editor.defineWhenExpression",
        "when": "inKeybindings && keybindingFocus"
    },
    {
        "key": "ctrl+k ctrl+i",
        "command": "-list.showHover",
        "when": "listFocus && !inputFocus && !treestickyScrollFocused"
    },
    {
        "key": "ctrl+k i",
        "command": "-notebook.cell.chat.start",
        "when": "config.notebook.experimental.cellChat && inlineChatHasProvider && notebookEditable && notebookEditorFocused && !inputFocus || config.notebook.experimental.generate && inlineChatHasProvider && notebookEditable && notebookEditorFocused && !inputFocus"
    },
    {
        "key": "ctrl+k f2",
        "command": "-togglePeekWidgetFocus",
        "when": "inReferenceSearchEditor || referenceSearchVisible"
    },
    {
        "key": "ctrl+k down",
        "command": "-views.moveViewDown",
        "when": "focusedView != ''"
    },
    {
        "key": "ctrl+k left",
        "command": "-views.moveViewLeft",
        "when": "focusedView != ''"
    },
    {
        "key": "ctrl+k right",
        "command": "-views.moveViewRight",
        "when": "focusedView != ''"
    },
    {
        "key": "ctrl+k up",
        "command": "-views.moveViewUp",
        "when": "focusedView != ''"
    },
    {
        "key": "ctrl+k ctrl+p",
        "command": "-workbench.action.showAllEditors"
    },
    {
        "key": "ctrl+k ctrl+0",
        "command": "-editor.foldAll",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+/",
        "command": "-editor.foldAllBlockComments",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+8",
        "command": "-editor.foldAllMarkerRegions",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+1",
        "command": "-editor.foldLevel1",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+2",
        "command": "-editor.foldLevel2",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+3",
        "command": "-editor.foldLevel3",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+4",
        "command": "-editor.foldLevel4",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+5",
        "command": "-editor.foldLevel5",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+6",
        "command": "-editor.foldLevel6",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+7",
        "command": "-editor.foldLevel7",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+[",
        "command": "-editor.foldRecursively",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+f",
        "command": "-editor.action.formatSelection",
        "when": "editorHasDocumentSelectionFormattingProvider && editorTextFocus && !editorReadonly"
    },
    {
        "key": "ctrl+k ctrl+r",
        "command": "-git.revertSelectedRanges",
        "when": "isInDiffEditor && !operationInProgress"
    },
    {
        "key": "ctrl+k ctrl+alt+s",
        "command": "-git.stageSelectedRanges",
        "when": "isInDiffEditor && !operationInProgress"
    },
    {
        "key": "ctrl+k ctrl+n",
        "command": "-git.unstageSelectedRanges",
        "when": "isInDiffEditor && !operationInProgress"
    },
    {
        "key": "ctrl+k ctrl+q",
        "command": "-workbench.action.navigateToLastEditLocation"
    },
    {
        "key": "ctrl+k ctrl+r",
        "command": "-workbench.action.keybindingsReference"
    },
    {
        "key": "ctrl+k i",
        "command": "-inlineChat.start",
        "when": "editorFocus && inlineChatHasProvider && !editorReadonly"
    },
    {
        "key": "ctrl+k v",
        "command": "-markdown.showPreviewToSide",
        "when": "!notebookEditorFocused && editorLangId == 'markdown'"
    },
    {
        "key": "ctrl+k ctrl+d",
        "command": "-editor.action.moveSelectionToNextFindMatch",
        "when": "editorFocus"
    },
    {
        "key": "ctrl+k ctrl+c",
        "command": "-notebook.cell.collapseCellInput",
        "when": "notebookCellListFocused && !inputFocus && !notebookCellInputIsCollapsed"
    },
    {
        "key": "ctrl+k t",
        "command": "-notebook.cell.collapseCellOutput",
        "when": "notebookCellHasOutputs && notebookCellListFocused && !inputFocus && !notebookCellOutputIsCollapsed"
    },
    {
        "key": "ctrl+k ctrl+c",
        "command": "-notebook.cell.expandCellInput",
        "when": "notebookCellInputIsCollapsed && notebookCellListFocused"
    },
    {
        "key": "ctrl+k t",
        "command": "-notebook.cell.expandCellOutput",
        "when": "notebookCellListFocused && notebookCellOutputIsCollapsed"
    },
    {
        "key": "ctrl+k ctrl+shift+\\",
        "command": "-notebook.cell.split",
        "when": "editorTextFocus && notebookCellEditable && notebookEditable && notebookEditorFocused"
    },
    {
        "key": "ctrl+k y",
        "command": "-notebook.cell.toggleOutputScrolling",
        "when": "notebookCellHasOutputs && notebookCellListFocused && !inputFocus"
    },
    {
        "key": "ctrl+k ctrl+shift+n",
        "command": "-notifications.showList"
    },
    {
        "key": "ctrl+k f12",
        "command": "-editor.action.revealDefinitionAside",
        "when": "editorHasDefinitionProvider && editorTextFocus && !isInEmbeddedEditor"
    },
    {
        "key": "ctrl+k ctrl+f12",
        "command": "-editor.action.revealDefinitionAside",
        "when": "editorHasDefinitionProvider && editorTextFocus && isWeb && !isInEmbeddedEditor"
    },
    {
        "key": "ctrl+k ctrl+t",
        "command": "-workbench.action.selectTheme"
    },
    {
        "key": "ctrl+k ctrl+s",
        "command": "-workbench.action.openGlobalKeybindings"
    },
    {
        "key": "ctrl+k ctrl+u",
        "command": "-editor.action.removeCommentLine",
        "when": "editorTextFocus && !editorReadonly"
    },
    {
        "key": "ctrl+k ctrl+.",
        "command": "-editor.removeManualFoldingRanges",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+k",
        "command": "-editor.action.selectFromAnchorToCursor",
        "when": "editorTextFocus && selectionAnchorSet"
    },
    {
        "key": "ctrl+k ctrl+b",
        "command": "-editor.action.setSelectionAnchor",
        "when": "editorTextFocus"
    },
    {
        "key": "ctrl+k ctrl+i",
        "command": "-editor.action.showHover",
        "when": "editorTextFocus"
    },
    {
        "key": "ctrl+k ctrl+i",
        "command": "-workbench.action.terminal.focusHover",
        "when": "terminalFocus && terminalHasBeenCreated || terminalFocus && terminalIsOpen || terminalFocus && terminalProcessSupported || terminalHasBeenCreated && terminalTabsFocus || terminalIsOpen && terminalTabsFocus || terminalProcessSupported && terminalTabsFocus"
    },
    {
        "key": "ctrl+k ctrl+l",
        "command": "-editor.toggleFold",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+x",
        "command": "-editor.action.trimTrailingWhitespace",
        "when": "editorTextFocus && !editorReadonly"
    },
    {
        "key": "ctrl+k ctrl+j",
        "command": "-editor.unfoldAll",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+=",
        "command": "-editor.unfoldAllExcept",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+9",
        "command": "-editor.unfoldAllMarkerRegions",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+]",
        "command": "-editor.unfoldRecursively",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+k ctrl+shift+w",
        "command": "-workbench.action.closeAllGroups"
    },
    {
        "key": "ctrl+k ctrl+w",
        "command": "-workbench.action.closeAllEditors"
    },
    {
        "key": "ctrl+k w",
        "command": "-workbench.action.closeEditorsInGroup"
    },
    {
        "key": "ctrl+k u",
        "command": "-workbench.action.closeUnmodifiedEditors"
    },
    {
        "key": "ctrl+k o",
        "command": "-workbench.action.copyEditorToNewWindow",
        "when": "activeEditor"
    },
    {
        "key": "ctrl+k ctrl+up",
        "command": "-workbench.action.focusAboveGroup"
    },
    {
        "key": "ctrl+k ctrl+down",
        "command": "-workbench.action.focusBelowGroup"
    },
    {
        "key": "ctrl+k ctrl+left",
        "command": "-workbench.action.focusLeftGroup"
    },
    {
        "key": "ctrl+k ctrl+right",
        "command": "-workbench.action.focusRightGroup"
    },
    {
        "key": "ctrl+k ctrl+shift+\\",
        "command": "-workbench.action.joinEditorInGroup",
        "when": "sideBySideEditorActive"
    },
    {
        "key": "ctrl+k enter",
        "command": "-workbench.action.keepEditor"
    },
    {
        "key": "ctrl+k down",
        "command": "-workbench.action.moveActiveEditorGroupDown"
    },
    {
        "key": "ctrl+k left",
        "command": "-workbench.action.moveActiveEditorGroupLeft"
    },
    {
        "key": "ctrl+k right",
        "command": "-workbench.action.moveActiveEditorGroupRight"
    },
    {
        "key": "ctrl+k up",
        "command": "-workbench.action.moveActiveEditorGroupUp"
    },
    {
        "key": "ctrl+k ctrl+pagedown",
        "command": "-workbench.action.nextEditorInGroup"
    },
    {
        "key": "ctrl+k ctrl+pageup",
        "command": "-workbench.action.previousEditorInGroup"
    },
    {
        "key": "ctrl+k shift+enter",
        "command": "-workbench.action.pinEditor",
        "when": "!activeEditorIsPinned"
    },
    {
        "key": "ctrl+k ctrl+\\",
        "command": "-workbench.action.splitEditorDown"
    },
    {
        "key": "ctrl+k shift+o",
        "command": "-workbench.action.compareEditor.openSide",
        "when": "inDiffEditor"
    },
    {
        "key": "ctrl+k p",
        "command": "-workbench.action.files.copyPathOfActiveFile"
    },
    {
        "key": "ctrl+k ctrl+o",
        "command": "-workbench.action.files.openLocalFolder",
        "when": "remoteFileDialogVisible"
    },
    {
        "key": "ctrl+k r",
        "command": "-workbench.action.files.revealActiveFileInWindows"
    },
    {
        "key": "ctrl+k d",
        "command": "-workbench.files.action.compareWithSaved"
    },
    {
        "key": "ctrl+k ctrl+alt+c",
        "command": "-copyFilePath",
        "when": "editorFocus"
    },
    {
        "key": "ctrl+k ctrl+shift+alt+c",
        "command": "-copyRelativeFilePath",
        "when": "editorFocus"
    },
    {
        "key": "ctrl+k ctrl+o",
        "command": "-workbench.action.files.openFolder",
        "when": "openFolderWorkspaceSupport"
    },
    {
        "key": "left",
        "command": "-list.collapse",
        "when": "listFocus && treeElementCanCollapse && !inputFocus && !treestickyScrollFocused || listFocus && treeElementHasParent && !inputFocus && !treestickyScrollFocused"
    },
    {
        "key": "ctrl+left",
        "command": "-list.collapseAll",
        "when": "listFocus && !inputFocus && !treestickyScrollFocused"
    },
    {
        "key": "left",
        "command": "-list.stickyScroll.collapse",
        "when": "treestickyScrollFocused"
    },
    {
        "key": "left",
        "command": "-notification.collapse",
        "when": "notificationFocus"
    },
    {
        "key": "ctrl+k",
        "command": "editor.foldAll"
    },
    {
        "key": "ctrl+shift+[",
        "command": "-editor.fold",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+shift+]",
        "command": "-editor.unfold",
        "when": "editorTextFocus && foldingEnabled"
    },
    {
        "key": "ctrl+l",
        "command": "editor.unfoldAll"
    },
    {
        "key": "right",
        "command": "-notebook.unfold",
        "when": "notebookEditorFocused && !inputFocus && activeEditor == 'workbench.editor.notebook'"
    },
    {
        "key": "ctrl+shift+]",
        "command": "-notebook.unfold",
        "when": "notebookEditorFocused && !inputFocus && activeEditor == 'workbench.editor.notebook'"
    },
    {
        "key": "ctrl+shift+[",
        "command": "-notebook.fold",
        "when": "notebookEditorFocused && !inputFocus && activeEditor == 'workbench.editor.notebook'"
    },
    {
        "key": "left",
        "command": "-notebook.fold",
        "when": "notebookEditorFocused && !inputFocus && activeEditor == 'workbench.editor.notebook'"
    },
    {
        "key": "ctrl+1",
        "command": "-workbench.action.focusFirstEditorGroup"
    },
    {
        "key": "ctrl+.",
        "command": "workbench.action.focusActiveEditorGroup"
    },
    {
        "key": "shift+alt+1",
        "command": "-workbench.action.moveEditorToFirstGroup"
    },
    {
        "key": "ctrl+u",
        "command": "-cursorUndo",
        "when": "textInputFocus"
    },
    {
        "key": "shift+alt+9",
        "command": "-workbench.action.moveEditorToLastGroup"
    },
    {
        "key": "alt+3 alt+k",
        "command": "workbench.scm.action.collapseAllRepositories"
    },
    {
        "key": "ctrl+shift+j",
        "command": "-workbench.action.search.toggleQueryDetails",
        "when": "inSearchEditor || searchViewletFocus"
    },
    {
        "key": "ctrl+i",
        "command": "-workbench.action.chat.startVoiceChat",
        "when": "chatIsEnabled && hasSpeechProvider && inChatInput && !chatSessionRequestInProgress && !editorFocus && !notebookEditorFocused && !scopedVoiceChatGettingReady && !speechToTextInProgress || chatIsEnabled && hasSpeechProvider && inlineChatFocused && !chatSessionRequestInProgress && !editorFocus && !notebookEditorFocused && !scopedVoiceChatGettingReady && !speechToTextInProgress"
    },
    {
        "key": "ctrl+i",
        "command": "-workbench.action.chat.stopListeningAndSubmit",
        "when": "inChatInput && voiceChatInProgress && scopedVoiceChatInProgress == 'editor' || inChatInput && voiceChatInProgress && scopedVoiceChatInProgress == 'inline' || inChatInput && voiceChatInProgress && scopedVoiceChatInProgress == 'quick' || inChatInput && voiceChatInProgress && scopedVoiceChatInProgress == 'view' || inlineChatFocused && voiceChatInProgress && scopedVoiceChatInProgress == 'editor' || inlineChatFocused && voiceChatInProgress && scopedVoiceChatInProgress == 'inline' || inlineChatFocused && voiceChatInProgress && scopedVoiceChatInProgress == 'quick' || inlineChatFocused && voiceChatInProgress && scopedVoiceChatInProgress == 'view'"
    },
    {
        "key": "ctrl+i",
        "command": "-inlineChat.start",
        "when": "editorFocus && inlineChatHasProvider && inlineChatPossible && !editorReadonly && !editorSimpleInput"
    },
    {
        "key": "ctrl+i",
        "command": "-inlineChat.startWithCurrentLine",
        "when": "inlineChatHasProvider && inlineChatShowingHint && !editorReadonly && !inlineChatVisible"
    },
    {
        "key": "ctrl+i",
        "command": "-workbench.action.terminal.chat.start",
        "when": "terminalChatAgentRegistered && terminalFocusInAny && terminalHasBeenCreated || terminalChatAgentRegistered && terminalFocusInAny && terminalProcessSupported"
    },
    {
        "key": "ctrl+i",
        "command": "-editor.action.triggerSuggest",
        "when": "editorHasCompletionItemProvider && textInputFocus && !editorReadonly && !suggestWidgetVisible"
    },
    {
        "key": "ctrl+i",
        "command": "-focusSuggestion",
        "when": "suggestWidgetVisible && textInputFocus && !suggestWidgetHasFocusedSuggestion"
    },
    {
        "key": "ctrl+i",
        "command": "-inlineChat.holdForSpeech",
        "when": "hasSpeechProvider && inlineChatHasProvider && inlineChatVisible && textInputFocus"
    },
    {
        "key": "ctrl+i",
        "command": "-notebook.cell.chat.start",
        "when": "config.notebook.experimental.cellChat && notebookChatAgentRegistered && notebookEditable && notebookEditorFocused && !inputFocus || config.notebook.experimental.generate && notebookChatAgentRegistered && notebookEditable && notebookEditorFocused && !inputFocus"
    },
    {
        "key": "ctrl+i",
        "command": "-toggleSuggestionDetails",
        "when": "suggestWidgetHasFocusedSuggestion && suggestWidgetVisible && textInputFocus"
    },
    {
        "key": "ctrl+i",
        "command": "-workbench.action.chat.holdToVoiceChatInChatView",
        "when": "chatIsEnabled && hasSpeechProvider && !chatSessionRequestInProgress && !editorFocus && !inChatInput && !inlineChatFocused && !notebookEditorFocused"
    },
    {
        "key": "ctrl+i",
        "command": "editor.action.goToImplementation",
        "when": "editorHasImplementationProvider && editorTextFocus"
    },
    {
        "key": "ctrl+f12",
        "command": "-editor.action.goToImplementation",
        "when": "editorHasImplementationProvider && editorTextFocus"
    },
    {
        "key": "ctrl+i",
        "command": "editor.action.revealDefinition",
        "when": "editorHasDefinitionProvider && editorTextFocus && isWeb"
    },
    {
        "key": "ctrl+f12",
        "command": "-editor.action.revealDefinition",
        "when": "editorHasDefinitionProvider && editorTextFocus && isWeb"
    },
    {
        "key": "ctrl+u",
        "command": "editor.action.goToReferences",
        "when": "editorHasReferenceProvider && editorTextFocus && !inReferenceSearchEditor && !isInEmbeddedEditor"
    },
    {
        "key": "shift+f12",
        "command": "-editor.action.goToReferences",
        "when": "editorHasReferenceProvider && editorTextFocus && !inReferenceSearchEditor && !isInEmbeddedEditor"
    },
    {
        "key": "ctrl+u",
        "command": "goToPreviousReference",
        "when": "inReferenceSearchEditor || referenceSearchVisible"
    },
    {
        "key": "shift+f12",
        "command": "-goToPreviousReference",
        "when": "inReferenceSearchEditor || referenceSearchVisible"
    },
    {
        "key": "ctrl+o",
        "command": "-workbench.action.files.openFile",
        "when": "true"
    },
    {
        "key": "ctrl+o",
        "command": "-workbench.action.files.openFolderViaWorkspace",
        "when": "!openFolderWorkspaceSupport && workbenchState == 'workspace'"
    },
    {
        "key": "ctrl+o",
        "command": "-workbench.action.files.openFileFolder",
        "when": "isMacNative && openFolderWorkspaceSupport"
    },
    {
        "key": "ctrl+o",
        "command": "-workbench.action.files.openLocalFile",
        "when": "remoteFileDialogVisible"
    },
    {
        "key": "ctrl+o",
        "command": "editor.action.revealDefinition",
        "when": "editorHasDefinitionProvider && editorTextFocus"
    },
    {
        "key": "f12",
        "command": "-editor.action.revealDefinition",
        "when": "editorHasDefinitionProvider && editorTextFocus"
    },
    {
        "key": "ctrl+p",
        "command": "-workbench.action.quickOpen"
    },
    {
        "key": "ctrl+p",
        "command": "-workbench.action.quickOpenNavigateNextInFilePicker",
        "when": "inFilesPicker && inQuickOpen"
    },
    {
        "key": "ctrl+p",
        "command": "change-case.changeCase"
    },
    {
        "key": "alt+q",
        "command": "-change-case.changeCase"
    },
    {
        "key": "alt+q",
        "command": "workbench.files.action.collapseExplorerFolders"
    },
    {
        "key": "ctrl+j",
        "command": "-workbench.action.togglePanel"
    },
    {
        "key": "alt+4 alt+k",
        "command": "variables.collapse",
        "when": "view == 'workbench.debug.variablesView'"
    },
    {
        "key": "ctrl+j",
        "command": "workbench.files.action.collapseExplorerFolders"
    },
    {
        "key": "ctrl+shift+e",
        "command": "-workbench.view.explorer",
        "when": "viewContainer.workbench.view.explorer.enabled"
    },
    {
        "key": "alt+1",
        "command": "-workbench.action.openEditorAtIndex1"
    },
    {
        "key": "alt+1",
        "command": "workbench.view.explorer"
    },
    {
        "key": "ctrl+shift+f",
        "command": "-workbench.view.search",
        "when": "workbench.view.search.active && neverMatch =~ /doesNotMatch/"
    },
    {
        "key": "alt+2",
        "command": "-workbench.action.openEditorAtIndex2"
    },
    {
        "key": "ctrl+shift+g",
        "command": "-workbench.view.scm",
        "when": "workbench.scm.active"
    },
    {
        "key": "ctrl+shift+g g",
        "command": "-workbench.view.scm",
        "when": "workbench.scm.active && !gitlens:disabled && config.gitlens.keymap == 'chorded'"
    },
    {
        "key": "alt+3",
        "command": "-workbench.action.openEditorAtIndex3"
    },
    {
        "key": "alt+3",
        "command": "workbench.view.scm"
    },
    {
        "key": "ctrl+shift+d",
        "command": "-workbench.view.debug",
        "when": "viewContainer.workbench.view.debug.enabled"
    },
    {
        "key": "alt+4",
        "command": "-workbench.action.openEditorAtIndex4"
    },
    {
        "key": "alt+4",
        "command": "workbench.view.debug"
    },
    {
        "key": "shift+alt+f",
        "command": "search.action.restrictSearchToFolder",
        "when": "folderMatchWithResourceFocus && searchViewletVisible"
    },
    {
        "key": "shift+alt+f",
        "command": "-search.action.restrictSearchToFolder",
        "when": "folderMatchWithResourceFocus && searchViewletVisible"
    },
    {
        "key": "alt+2",
        "command": "filesExplorer.findInFolder",
        "when": "explorerResourceIsFolder && filesExplorerFocus && foldersViewVisible && !inputFocus"
    },
    {
        "key": "shift+alt+f",
        "command": "-filesExplorer.findInFolder",
        "when": "explorerResourceIsFolder && filesExplorerFocus && foldersViewVisible && !inputFocus"
    },
    {
        "key": "alt+6",
        "command": "-workbench.action.openEditorAtIndex6"
    }
]


```

### read debug envs

خیلی وقتا کپی env  ها توی launch.sjon vscode  هم باید باشه ، و سینک بودن اینا خیلی درد سره ، با این اسکریپت  از دیباگ می خونیم 

```sh

#!/usr/bin/env bash
# run.sh - read envs from nearest .vscode/launch.json and run go run main.go
# Requires: bash, jq
set -euo pipefail

# -------------------------
# Configuration / Defaults
# -------------------------
CONF_NAME=""   # optional: set with --name "Config Name"
# -------------------------

die() { echo "ERROR: $*" >&2; exit 1; }

# check dependencies
if ! command -v jq >/dev/null 2>&1; then
  die "this script requires 'jq'. Install it (e.g. apt install jq) and retry."
fi

# find nearest .vscode/launch.json by walking up
find_launch() {
  local cur="$PWD"
  while true; do
    if [ -f "$cur/.vscode/launch.json" ]; then
      printf '%s' "$cur/.vscode/launch.json"
      return 0
    fi
    [ "$cur" = "/" ] && break
    cur=$(dirname "$cur")
  done
  return 1
}

LAUNCH="$(find_launch)" || die ".vscode/launch.json not found in this or parent directories"

# Build a function that emits key\0value\0 pairs (NUL-separated)
emit_key_val_nul() {
  # remove lines that start (possibly after whitespace) with //
  if [ -n "$CONF_NAME" ]; then
    sed -E 's/^[[:space:]]*\/\/.*$//' "$LAUNCH" \
      | jq -r --arg name "$CONF_NAME" '
          (.configurations[] | select(.name == $name) | .env // {})
          | to_entries[]
          | "\(.key|tostring|gsub("^\\s+|\\s+$";""))\u0000\(.value|tostring|gsub("^\\s+|\\s+$";""))\u0000"
        ' -
  else
    sed -E 's/^[[:space:]]*\/\/.*$//' "$LAUNCH" \
      | jq -r '
          (.configurations[0].env // {})
          | to_entries[]
          | "\(.key|tostring|gsub("^\\s+|\\s+$";""))\u0000\(.value|tostring|gsub("^\\s+|\\s+$";""))\u0000"
        ' -
  fi
}

# Load envs into this shell
while IFS= read -r -d '' key && IFS= read -r -d '' val; do
  key="$(echo -n "$key" | xargs)"   # trim extra whitespace, just in case
  val="$(echo -n "$val" | xargs)"
  if [[ "$key" =~ ^[A-Za-z_][A-Za-z0-9_]*$ ]]; then
    export "$key"="$val"
  else
    echo "⚠️  Skipping invalid env key: '$key'"
  fi
done < <(emit_key_val_nul)

echo "✅ Loaded environment variables from: $LAUNCH"
echo "🚀 Running: go run main.go"
echo

exec go run main.go


```


### dynamic title merge request by last commit :

وقتی یه برنچ جدید میسازیم ، تایتل اون همیشه اولین کامیت هست و باید دستی تغییرش بدیم ، ولی با این اسکریپت می شه همیشه متنش رو متناسب با آخرین کامیت ، آپدیت کرد 

```bash

#!/usr/bin/env bash
set -euo pipefail

API="https://git.srxx.org/api/v4"
TOKEN="${GITLAB_TOKEN:-}"

# --- Validation ---
if [[ -z "$TOKEN" ]]; then
  echo "❌ GITLAB_TOKEN is not set"
  exit 1
fi

if ! command -v jq >/dev/null 2>&1; then
  echo "❌ jq not installed"
  exit 1
fi

# --- Detect project path from git remote ---
REMOTE_URL=$(git remote get-url origin 2>/dev/null || echo "")
if [[ -z "$REMOTE_URL" ]]; then
  echo "❌ Could not detect git remote"
  exit 1
fi

# Extract project path (works for SSH and HTTPS)
case "$REMOTE_URL" in
  ssh://git@*/* ) PROJECT_PATH="${REMOTE_URL#ssh://git@*/}" ;;
  git@*:* )      PROJECT_PATH="${REMOTE_URL#*:}" ;;
  http*://* )    PROJECT_PATH="${REMOTE_URL#*git.srxx.org/}" ;;
  * )            echo "❌ Unrecognized remote URL format: $REMOTE_URL"; exit 1 ;;
esac

PROJECT_PATH="${PROJECT_PATH%.git}"
echo "🔍 Searching project: $PROJECT_PATH"

# --- Get project ID dynamically ---
PROJECT_ID=$(curl -s --header "PRIVATE-TOKEN: $TOKEN" \
  "$API/projects?search=$(basename $PROJECT_PATH)" | jq -r \
  ".[] | select(.path_with_namespace==\"$PROJECT_PATH\") | .id")

if [[ -z "$PROJECT_ID" ]]; then
  echo "❌ Could not find project ID for '$PROJECT_PATH'"
  exit 1
fi

echo "✅ Found project ID: $PROJECT_ID"

# --- Fetch info ---
BRANCH=$(git rev-parse --abbrev-ref HEAD)
LAST_COMMIT_MSG=$(git log -1 --pretty=%s)

# --- Fetch merge requests for current branch ---
RESPONSE=$(curl -s --header "PRIVATE-TOKEN: $TOKEN" \
  "$API/projects/$PROJECT_ID/merge_requests?state=all&source_branch=$BRANCH")

MR_IID=$(echo "$RESPONSE" | jq -r '.[0].iid // empty')

if [[ -z "$MR_IID" ]]; then
  echo "❌ No merge request found for branch '$BRANCH'"
  echo "🔍 Raw API output:"
  echo "$RESPONSE"
  exit 1
fi

# --- Update MR title ---
curl -s --request PUT \
  --header "PRIVATE-TOKEN: $TOKEN" \
  --data-urlencode "title=$LAST_COMMIT_MSG" \
  "$API/projects/$PROJECT_ID/merge_requests/$MR_IID" >/dev/null

echo "✅ Updated MR !$MR_IID title → \"$LAST_COMMIT_MSG\""



```
# bootstrap os
### env

```
ZSH=/home/mo30/.oh-my-zsh
GOPATH=/home/mo30/go
GOPRIVATE=git.srxx.org
GITLAB_TOKEN=glpat-***
SHELL=/usr/bin/zsh

```

### zsh

+ which zsh

`/usr/bin/zsh`


+  nano ~/.zshrc

```
export ZSH="$HOME/.oh-my-zsh"
ZSH_THEME="robbyrussell"
source $ZSH/oh-my-zsh.sh
plugins=(
  git
  docker
  ssh-agent
  z
  sudo
  history
  extract
  colored-man-pages
)

```

+ در نهایت برای اعمال

`source ~/.zshrc`

### tmux

### gitlab

+ اول بهتره کلید گیت با کلید ssh  ایمیل یکی داشته باشن

```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
ssh-keygen -t ed25519 -C "you@example.com"
```

+ باید توی گیت لب این رو اضافه کنید 

`cat ~/.ssh/id_ed25519.pub`


+ حتما سطح دسترسی رو تنظیم کنید

```
sudo chown mo30:mo30 /home/mo30/.ssh/config
sudo chmod 600 /home/mo30/.ssh/config
sudo chmod 700 /home/mo30/.ssh    
sudo chmod 600 /home/mo30/.ssh/id_rsa   
sudo chmod 644 /home/mo30/.ssh/id_rsa.pub
```


+ اگر ssh  بزنید باید پاسخ بگیرید :

`ssh -T  git@git.srxx.org `

Welcome to GitLab, @mo30.developer!




+ باید توی کافیگ اضافه کنید

`nano ~/.ssh/config`

```

Host git.srxx.org
    HostName 157.180.85.51
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ProxyCommand /usr/bin/nc -x 127.0.0.1:1080 %h %p

```

+ کانفیگ گیت 

`cat ~/.gitconfig`

```
[user]
        name = mo30
        email = mo30.developer@gmail.com
        signingkey = ******
[commit]
        gpgsign = true
[url "ssh://git@git.srxx.org/"]
        insteadOf = https://git.srxx.org/
```
 
 ### music
+ Audacious

موزیک پلیر خوب


+ sudo apt install playerctl

+ sudo apt install pulseaudio-utils -y

میشه با این پکیج کوچیک hot key  اضافه کرد

و در نهایت باید توی کاستوم ها ، اینا رو ست کنیم

`dconf load /org/gnome/settings-daemon/plugins/media-keys/ < media-keys-backup.dconf`

فایل زیر باید لود بشه 
```
[/]
custom-keybindings=['/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/', '/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom1/', '/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom2/', '/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom3/', '/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom4/', '/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom5/', '/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom6/']

[custom-keybindings/custom0]
binding='<Control><Alt>KP_Multiply'
command='playerctl -p audacious play-pause'
name='Pause Audacious'

[custom-keybindings/custom1]
binding='<Control><Alt>KP_Add'
command='playerctl -p audacious next\n'
name='Next Audacious'

[custom-keybindings/custom2]
binding='<Control><Alt>KP_Subtract'
command='playerctl -p audacious previous'
name='prevus Audacious'

[custom-keybindings/custom3]
binding='<Control><Alt>KP_Left'
command='playerctl -p audacious position 10-'
name='Rewind 10s'

[custom-keybindings/custom4]
binding='<Control><Alt>KP_Right'
command='playerctl -p audacious position 10+'
name='Forward 10s'

[custom-keybindings/custom5]
binding='<Control><Alt>KP_Up'
command='pactl set-sink-volume @DEFAULT_SINK@ +5%'
name='Volume Up'

[custom-keybindings/custom6]
binding='<Control><Alt>KP_Down'
command='pactl set-sink-volume @DEFAULT_SINK@ -5%'
name='Volume Down'
```
### Ubuntu 24.04 — Wayland / Xwayland resize bug:

اینو تو zsh  بیفزای

`alias gg='pkill -HUP -f mutter-x11-frames || pkill -f mutter-x11-frames'`

### clipboard


+ diodon
