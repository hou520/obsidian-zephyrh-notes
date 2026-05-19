![[Pasted image 20260512144905.png]]

配置方式：
Ctrl+Shift+P
>Preference Open User Settings(JSON)

```
{

    "window.commandCenter": true,

    "update.releaseTrack": "dev",

    "update.mode": "silentlyApplyOnQuit",

    "files.autoSave": "afterDelay",

    "workbench.settings.applyToAllProfiles": [

        "workbench.activityBar.location",

        "java.configuration.maven.globalSettings"

    ],

    "workbench.activityBar.orientation": "vertical",

    "java.configuration.maven.userSettings": "D:\\Java\\apache-maven-3.6.3\\conf\\settings.xml",

    "java.configuration.maven.globalSettings": "D:\\Java\\apache-maven-3.6.3\\conf\\settings.xml",

    "git.autofetch": true,

    "editor.fontLigatures": false,

    "resharper.region.region": "China",

    "resharper.dataSharing.allowDataSharing": false,

    "workbench.colorCustomizations": {

        // ======================

        // 1 AI返回内容区域

        // ======================

        // "editor.background": "#0f172a",

        // "editor.foreground": "#e5e7eb",

        // "editorWidget.background": "#111827",

        // ======================

        // 2 Chat Markdown渲染区域

        // ======================

        // "textBlockQuote.background": "#cc1357",

        // "textCodeBlock.background": "#2043dd",

        // ======================

        // 3 左侧 & 下方面板

        // ======================

        // "sideBar.background": "#020617",

        // "panel.background": "#020617",

        // ======================

        // 4 Agent输入框

        // ======================

        "input.background": "#08195b",

        "input.foreground": "#e5e7eb",

        "input.border": "#374151",

        // ======================

        // 5 滚动条

        // ======================

        // "scrollbarSlider.background": "#374151",

        // "scrollbarSlider.hoverBackground": "#4b5563",

        // ======================

        // 6 活动栏

        // ======================

        // "activityBar.background": "#020617"

    },

    "workbench.colorTheme": "Default Dark+",

    "editor.wordWrap": "on",

    "window.autoDetectColorScheme": false,

}
```