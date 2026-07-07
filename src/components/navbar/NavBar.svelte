<script lang="ts">
  import type { NavItemType } from "./NavTypes";
  import { onMount } from "svelte";
  import { initTheme, toggleThemeWithTransition } from "./helpers/theme";
  import LeftNavBtn from "./LeftNavBtn.svelte";
  import MenuBar from "./MenuBar.svelte";
  import RightNavBar from "./RightNavBar.svelte";
  import { t } from "@/i18n";

  interface Props {
    name: string;
    navLinks?: NavItemType[];
    clickToggleCallback?: (state: boolean) => void;
    onSearch?: () => void;
  }

  const {
    name,
    navLinks = [],
    clickToggleCallback = () => {},
    onSearch = () => {},
  }: Props = $props();

  let showNav = $state(true);
  let atTop = $state(true);
  let isDark = $state(false);

  onMount(() => {
    if (typeof window === "undefined" || typeof document === "undefined")
      return;

    // Initialize theme based on storage or system preference
    const currentTheme = initTheme(document, window);
    isDark = currentTheme === "dark";

    let lastScroll = window.scrollY;
    atTop = lastScroll <= 0;
    let ticking = false;

    const handleScroll = () => {
      if (ticking) return;

      ticking = true;
      window.requestAnimationFrame(() => {
        const current = window.scrollY;
        if (current > lastScroll) showNav = false;
        else if (current < lastScroll) showNav = true;

        lastScroll = current;
        atTop = current <= 0;
        ticking = false;
      });
    };

    window.addEventListener("scroll", handleScroll, { passive: true });
    return () => window.removeEventListener("scroll", handleScroll);
  });

  const handleToggleTheme = () => {
    if (typeof document === "undefined" || typeof window === "undefined")
      return;

    const next = toggleThemeWithTransition(
      document,
      window,
      isDark ? "dark" : "light",
    );
    isDark = next === "dark";
  };

  const handleSearch = () => onSearch?.();
  const randomPostLabel = t("random.title");
</script>

<nav
  id="nav"
  aria-label="主导航"
  class={`h-12.5 fixed top-0 w-full z-9 backdrop-blur-8 backdrop-saturate-180 ${atTop ? "nav-top" : "nav-bg"}`.trim()}
  style={showNav ? "" : "transform: translateY(-100%);"}
>
  <div
    class="mb-0 ml-auto mr-auto mt-0 flex flex-nowrap h-full w-[calc(100%-0.625rem)] w-85%"
  >
    <LeftNavBtn clickCallback={clickToggleCallback} />
    <MenuBar {name} {navLinks} />
    <RightNavBar>

      <li>
        <button
          type="button"
          class="nav-action text-5 border-none bg-transparent"
          onclick={handleToggleTheme}
          aria-label="Toggle theme"
        >
          <div class={isDark ? "i-ri-moon-line" : "i-ri-sun-line"}></div>
        </button>
      </li>
      <li>
        <button
          type="button"
          id="search"
          class="nav-action text-5 border-none bg-transparent"
          onclick={handleSearch}
          aria-label="Search"
        >
          <div class="i-ri-search-line"></div>
        </button>
      </li>
    </RightNavBar>
  </div>
</nav>

<style>
  #nav {
    transition: transform 0.4s ease;
  }

  .nav-bg {
    background-image: var(--nav-bg);
    box-shadow: 0.1rem 0.1rem 0.2rem var(--grey-9-a1);
    text-shadow: 0 0 0.0625rem var(--grey-9-a1);
    color: var(--text-color);
  }

  .nav-top {
    color: var(--header-text-color);
    background-image: linear-gradient(
      180deg,
      color-mix(in oklch, var(--grey-9) 45%, transparent) 0%,
      var(--grey-9-a2) 45%,
      var(--grey-1-a0) 100%
    );
    text-shadow: 0 0.125rem 0.25rem var(--grey-9-a5);
  }

  .nav-action {
    padding: 0.625rem 0.5rem 0.625rem;
    cursor: pointer;
    color: inherit;
    text-decoration: none;
    display: inline-flex;
    align-items: center;
    justify-content: center;
  }
</style>
