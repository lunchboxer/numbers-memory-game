<script>
    import { createEventDispatcher } from "svelte";
    import MoonIcon from "./moon-icon.svelte";
    import SunIcon from "./sun-icon.svelte";
    import LanguageIcon from "./language-icon.svelte";

    let prefersDark = window.matchMedia("(prefers-color-scheme: dark)"); // boolean
    let darkMode = prefersDark.matches;

    function toggleDarkMode() {
        darkMode = !darkMode;
        if (prefersDark.matches) {
            document.body.classList.toggle("light-theme");
        } else {
            document.body.classList.toggle("dark-mode");
        }
    }

    const dispatch = createEventDispatcher();

    function handleLanguageChange() {
        dispatch("language-change");
    }
</script>

<footer>
    <p>made with 💖 by james | <span class="icp">湘ICP备20004695号-1</span></p>
    <div class="button-group">
        <button class="clear-button" on:click={toggleDarkMode}>
            {#if darkMode}
                <MoonIcon />
            {:else}
                <SunIcon />
            {/if}
        </button>
        <button class="clear-button" on:click={handleLanguageChange}>
            <LanguageIcon />
        </button>
    </div>
</footer>

<style>
    footer {
        display: flex;
        justify-content: space-between;
        padding: 0.4rem 1rem;
        font-size: 0.8rem;
    }
    .icp {
        font-size: 0.6rem;
    }
    @media screen and (max-width: 600px) {
        footer {
            font-size: 0.7rem;
        }
    }

    .clear-button {
        padding: 0;
        margin: 0;
        background: none;
        color: var(--text-color);
        border: none;
        outline: none;
        cursor: pointer;
    }
    .button-group button {
        margin: 0 0.5rem;
    }
</style>
