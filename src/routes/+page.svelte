{#if loading}
    <div class="loading"
         transition:fade></div>
{/if}
<header>
    <img src="./logo.png"
         alt={tAppName}>
    <p>{tAppName}</p>
    <button class="close"
            on:click={quitPlugin}>
        <CloseOutline/>
    </button>
</header>
<div class="body {theme}">
    <button class="select"
            class:empty={!selectFiles.length}
            on:click={selectFile}
            disabled={buttonStatus === BUTTON_STATUS.LOADING}>
        {#if selectFiles.length === 1}
            <p>&lrm;{selectFileName}</p>
        {:else if selectFiles.length > 1}
            <p>&lrm;{tCount.replace("{COUNT}",selectFiles.length.toString())}</p>
        {:else}
            <p>{tSelectFile}</p>
        {/if}
    </button>
    <button class="submit"
            class:empty={!selectFiles.length}
            on:click={submit}
            disabled={!selectFiles.length || buttonStatus !== BUTTON_STATUS.DEFAULT}>
        <span class="placeholder">{tChange}</span>
        {#if buttonStatus === BUTTON_STATUS.DEFAULT}
            <p transition:fly>{tChange}</p>
        {:else if buttonStatus === BUTTON_STATUS.COMPLETE}
            <p transition:fly><CheckCircleSolid color="green"/></p>
        {:else if buttonStatus === BUTTON_STATUS.FAIL}
            <p transition:fly><CloseCircleSolid color="red"/></p>
        {/if}
        {#if buttonStatus === BUTTON_STATUS.LOADING}
            <img class="loading"
                 src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGQAAABkCAMAAABHPGVmAAAAS1BMVEUAAAAjIyMjIyMjIyMjIyMjIyOgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKDG167mAAAAGHRSTlMAMgkfFSbzLheZeOcjs8WlUArh0rtEcGZXAxgUAAACpklEQVRo3rSVWXaDMBAEx0I7i9kMvv9Jky8PonEiWeO6QL3uHgFlo3RjzO2Xp7XzsoY4ehJFNzfm+aLtoyMRFAtYwsyh3qNNamAJs0RfGwIliL27agVKkP4jTapACabxxVvcAJCAZqMizC1DggxdbQyUIHavXMM0Wikimnznxi0M9lJznz6uymhFgIvrhWjJqExdGegd0z6ApXXljob+xgV7HuZR6EAF4oMFS4HDUB7dCpZsh6Jsxja1vN8FmirA9+n6XdbtaiokJsssU8YbVFTMI7Hc6QINjmJcMsz+3yD0Gd18HB9nMTUOthyzDFBWVVeMO+6yUYqAA9e3/u1laaoi4oXhU2+okh4ePgahWvxh/P46iKJqxifjOAiUVV0YrgJlVdJZPDANQWoJHCUKBUE8R5l5dg4iHsWdZycpHEvCqS1DYgynvuCDIsGe9qWhLQkmm9xXA22JsPKnRb4t/Bi3iUQRI3lf/vhKSBQeZSQNk4gfcaQGnrv4ow9kYHchtpdkJSO9O/66Fjwu+fNqDxKSxfOP63uS6QsShF/jT7dmsIMwDMNQEjv//8sgcViEhTQV+4JvPSxPSbuui5uDrMh/CWkvoxdk8pDJbSu8ILENcgeObfW7RLmP1l5RexBaXLmDBBdEjkT+KYEgA1NCGQaqlTtwzwcE1zhRLcR+giAvRvlT0ZCUVIyJULllTyTQLNB4mbaHhlN0WYuFVCsKkoi/qca6hFB7sGtJ+CZKLdHeslXG+JvPGiTTRu/aot0QUAbs1oaWYvwmjbqTP9hNjke7DpOBMMwWoCKq3WamIqq9tuyDU8pwGszNKdVYrPKXuokLcLZWWMei5fqCSksVTYann+r7gu1yzHdEG+7g+BAqZuZC04kkoSK+AOiIrtfh3poB72fwBI1lTPajlg1tAAAAAElFTkSuQmCC"
                 alt="Loading">
        {/if}
    </button>
</div>

<script lang="ts">
    import { onMount } from "svelte";
    import { fade, fly } from "svelte/transition";
    import { DotLottie } from "@dotlottie/dotlottie-js";
    import {
        CloseOutline,
        CheckCircleSolid,
        CloseCircleSolid
    } from "flowbite-svelte-icons";
    import { setTimeout } from "node:timers/promises";
    import path from "node:path";
    import fs from "node:fs";

    enum BUTTON_STATUS {
        DEFAULT,
        LOADING,
        COMPLETE,
        FAIL
    }

    const THEME_COLOR = {
        light: ["#f8f8f9", "#dfdfe0", "#888a95", "#2c2f32"],
        lightgray: ["#dddee1", "#c7c7ca", "#6e8086", "#2c2f32"],
        gray: ["#3b3c40", "#515255", "#94969c", "#f8f9fb"],
        dark: ["#1f2023", "#363739", "#767b8a", "#f8f9fb"],
        blue: ["#151d36", "#2c344b", "#40475d", "#f8f9fb"],
        purple: ["#231b2b", "#393240", "#7a748e", "#f8f9fb"]
    }

    let tAppName = "";
    let tSelectFile = "";
    let tChange = "";
    let tCount = "";

    let buttonStatus = BUTTON_STATUS.DEFAULT;

    let selectFiles: string[] = [];
    let selectFileName = "";

    let theme: keyof typeof THEME_COLOR = "light";
    let loading = true;

    onMount(async () => {
        if(eagle.app.theme === "Auto") {
            theme = eagle.app.isDarkColors() ? "gray" : "light";
        } else {
            theme = eagle.app.theme.toLowerCase();
        }
        themeChange();
        eagle.onThemeChanged((value: string) => {
            theme = value.toLowerCase() as keyof typeof THEME_COLOR;
            themeChange();
        });
        await pluginTranslateInitialization();
        tAppName = i18next.t("manifest.app.name");
        tSelectFile = i18next.t("page.select");
        tChange = i18next.t("page.change");
        tCount = i18next.t("page.count");
        await eagle.window.setOpacity(1);
        loading = false;
    });

    const pluginTranslateInitialization = async () => {
        if(i18next.t("manifest.app.name")) {
            return true;
        } else {
            await setTimeout(50);
            await pluginTranslateInitialization();
        }
    }

    const quitPlugin = () => {
        window.close();
    }

    const themeChange = () => {
        document.documentElement.style.setProperty("--theme-background-color", THEME_COLOR[theme][0]);
        document.documentElement.style.setProperty("--theme-border-color", THEME_COLOR[theme][1]);
        document.documentElement.style.setProperty("--theme-control-color", THEME_COLOR[theme][2]);
        document.documentElement.style.setProperty("--theme-font-color", THEME_COLOR[theme][3]);
    }

    const selectFile = async () => {
        const result = await eagle.dialog.showOpenDialog({
            filters: [
                {
                    name: "Lottie JSON",
                    extensions: ["json"]
                }
            ],
            properties: [
                "openFile",
                "multiSelections"
            ]
        });
        if(result.canceled) {
            return;
        }
        selectFiles = result.filePaths;
        selectFileName = selectFiles.length === 1 ? path.basename(selectFiles[0]) : "";
        if(buttonStatus === BUTTON_STATUS.COMPLETE || buttonStatus === BUTTON_STATUS.FAIL) {
            buttonStatus = BUTTON_STATUS.DEFAULT;
        }
    }

    const submit = async () => {
        buttonStatus = BUTTON_STATUS.LOADING;
        try {
            for(const url of selectFiles) {
                const dotLottie = new DotLottie();
                const id = path.basename(url).split(".")[0];
                const value = await dotLottie.addAnimation({
                    id,
                    url,
                    autoplay: true
                }).build();
                await fs.promises.writeFile(path.join(path.dirname(url), `${id}.lottie`), await value.toBase64(), "base64");
            }
            buttonStatus = BUTTON_STATUS.COMPLETE;
        } catch(err: any) {
            console.error(err);
            buttonStatus = BUTTON_STATUS.FAIL;
        }
    }
</script>

<style lang="less">
    @import "../base.module";

    :root {
        --theme-background-color: #f8f9fb;
        --theme-border-color: #e3e5e7;
        --theme-control-color: #a0a1a4;
        --theme-font-color: #323339;
    }

    :global(*) {
        padding: 0;
        margin: 0;
        border: none;
        outline: 0;
        zoom: 1;
        resize: none;
        -webkit-text-size-adjust: none;
        font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", "微软雅黑", Arial, "-apple-system", sans-serif;
        -webkit-font-smoothing: antialiased;
        text-rendering: optimizeLegibility;
        -webkit-tap-highlight-color: rgba(0,0,0,0);
        -webkit-touch-callout: none;
        box-sizing: border-box;
        user-select: none;
    }
    :global(html),
    :global(body) {
        width: 100%;
        height: 100%;
    }
    :global(img) {
        display: block;
    }

    :global(#svelte) {
        display: flex;
        flex-direction: column;
        position: relative;
        overflow: hidden;
        width: 100%;
        height: 100%;
        background-color: var(--theme-background-color);
        transition: background-color ease .3s;
    }

    .loading {
        z-index: 1000;
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: var(--theme-background-color);
    }

    header {
        .align(v-center);
        position: relative;
        width: 100%;
        height: 40px;
        padding: 10px;
        border-bottom: 1px solid var(--theme-border-color);
        -webkit-app-region: drag;
        font-weight: bold;
        font-size: 14px;
        transition: border ease .3s;
        > img {
            height: 100%;
            margin-right: 10px;
        }
        > p {
            flex: 1;
            width: 0;
            color: var(--theme-font-color);
            transition: color ease .3s;
        }
        .close {
            .align(center);
            width: 20px;
            height: 100%;
            border-radius: 4px;
            background-color: transparent;
            color: var(--theme-font-color);
            cursor: pointer;
            transition: none ease .3s;
            transition-property: background-color, color;
            -webkit-app-region: no-drag;
            &:hover {
                background-color: var(--theme-border-color);
            }
            :global(> svg) {
                width: 16px;
                height: 16px;
            }
        }
    }

    .body {
        display: flex;
        flex: 1;
        width: 100%;
        height: 0;
        padding: 10px;
        &.light,
        &.lightgray {
            .select,
            .submit {
                &::before {
                    background-color: var(--theme-background-color);
                    filter: brightness(98%);
                }
                &:not(:disabled):hover::before {
                    background-color: rgba(100,100,100,.1);
                }
            }
        }
        &.gray,
        &.dark,
        &.blue,
        &.purple {
            .select,
            .submit {
                &::before {
                    background-color: var(--theme-background-color);
                    filter: brightness(85%);
                }
                &:not(:disabled):hover::before {
                    background-color: rgba(160,160,160,.1);
                }
            }
        }

        .select {
            .align(v-center);
            position: relative;
            flex: 1;
            width: 0;
            height: 100%;
            overflow: hidden;
            padding: 0 10px;
            border-radius: 5px;
            background-color: transparent;
            border: 1px solid var(--theme-border-color);
            font-size: 14px;
            cursor: pointer;
            transition: none ease .3s;
            transition-property: border, color;
            &::before {
                content: "";
                position: absolute;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
                transition: background-color ease .3s;
            }
            &:disabled {
                cursor: not-allowed;
            }
            &.empty {
                color: var(--theme-control-color);
            }
            > p {
                .overflow();
                z-index: 5;
                position: relative;
                direction: rtl;
                color: var(--theme-font-color);
                transition: color ease .3s;
            }
        }
        .submit {
            .align(v-center, inline-flex);
            position: relative;
            height: 100%;
            padding: 0 20px;
            margin-left: 10px;
            border-radius: 5px;
            overflow: hidden;
            background-color: transparent;
            border: 1px solid var(--theme-border-color);
            font-size: 14px;
            cursor: pointer;
            transition: border ease .3s;
            &::before {
                content: "";
                position: absolute;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
                transition: background-color ease .3s;
            }
            &:disabled {
                cursor: default;
            }
            &.empty {
                cursor: not-allowed;
            }
            .placeholder {
                opacity: 0;
            }
            > p {
                .absolute(cm);
                .align(center);
                z-index: 5;
                width: 100%;
                height: 100%;
                font-size: 14px;
                color: var(--theme-font-color);
                transition: color ease .3s;
                :global(> svg) {
                    width: 20px;
                    height: 20px;
                }
            }
            .loading {
                .absolute(cm);
                width: 20px;
                height: 20px;
                animation: rotate 1.2s linear infinite;
            }
            @keyframes rotate {
                from {
                    transform: rotate(0deg);
                }
                to {
                    transform: rotate(360deg);
                }
            }
        }
    }
</style>
