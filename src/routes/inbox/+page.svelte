<script lang="ts">
    import { base } from "$app/paths";
    import { onMount } from "svelte";
    import PostalMime from "postal-mime";

    // const endpoint = "http://localhost:3000";
    const endpoint = "https://test.opentrust.it";

    let isMobileMenuOpen = $state(false);
    let isUserMenuOpen = $state(false);
    let isSelectedEmailMenuOpen = $state(false);
    let emails = $state(new Map<number, any>());
    let page = $state(1);
    let selectedEmailId: number | null = $state(null);
    let parsedEmail: any = $state(null);
    let selectedPreviewLayout: "desktop" | "tablet" | "mobile" =
        $state("desktop");
    let selectedEmailViewMode: "html" | "text" | "raw" | "headers" =
        $state("html");

    function toggleMobileMenu() {
        isMobileMenuOpen = !isMobileMenuOpen;
    }

    function toggleUserMenu() {
        isUserMenuOpen = !isUserMenuOpen;
    }

    function toggleSelectedEmailMenu() {
        isSelectedEmailMenuOpen = !isSelectedEmailMenuOpen;
    }

    function setPreviewLayout(layout: "desktop" | "tablet" | "mobile") {
        selectedPreviewLayout = layout;
    }

    function setEmailViewMode(mode: "html" | "text" | "raw" | "headers") {
        if (mode === "text" && (!parsedEmail || !parsedEmail.text)) {
            return;
        }
        selectedEmailViewMode = mode;
        updateShownContent();
    }

    function signOut() {
        localStorage.removeItem("user");
        window.location.href = `${base}/`;
    }

    function downloadAttachment(attachment: any) {
        const blob = new Blob([attachment.content], {
            type: attachment.mimeType,
        });
        const url = URL.createObjectURL(blob);
        const a = document.createElement("a");
        a.href = url;
        a.target = "_blank";
        a.download = attachment.filename;
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
    }

    function formatEmails(emails: any[]) {
        return emails.map((email) => {
            if (email.name) {
                return `${email.name} <${email.address}>`;
            } else {
                return email.address;
            }
        });
    }

    async function parseEmail(emailContent: string) {
        const parsedEmail = await PostalMime.parse(emailContent);
        // console.log(parsedEmail);
        return parsedEmail;
    }

    function formatDate(dateString: string, full: boolean = false): string {
        const locale = "it-IT";
        const timeZone = "Europe/Rome";

        if (full) {
            const dateFormatter = new Intl.DateTimeFormat(locale, {
                year: "numeric",
                month: "numeric",
                day: "numeric",
                hour: "numeric",
                minute: "numeric",
                timeZone,
            });
            return dateFormatter.format(new Date(dateString));
        }

        const date = new Date(dateString);
        const today = new Date();

        // Check if the date is today
        const isToday =
            date.getDate() === today.getDate() &&
            date.getMonth() === today.getMonth() &&
            date.getFullYear() === today.getFullYear();

        if (isToday) {
            const timeFormatter = new Intl.DateTimeFormat(locale, {
                hour: "numeric",
                minute: "numeric",
                timeZone,
            });
            return timeFormatter.format(date);
        } else {
            const dateFormatter = new Intl.DateTimeFormat(locale, {
                year: "numeric",
                month: "numeric",
                day: "numeric",
                timeZone,
            });
            return dateFormatter.format(date);
        }
    }

    function escapeHTML(text: string) {
        const element = document.createElement("div");
        if (text) {
            element.innerText = text; // Set the text content
            element.textContent = text; // For browser compatibility
        }
        return element.innerHTML; // Get the escaped HTML content
    }

    function updateShownContent() {
        const shadowEmail = document.getElementById("shadow-email");
        if (!shadowEmail) return;
        const shadow =
            shadowEmail.shadowRoot ||
            shadowEmail.attachShadow({ mode: "open" });
        // clear the shadow root
        shadow.innerHTML = "";
        let shadowEmailContent: any = document.createElement("div");

        switch (selectedEmailViewMode) {
            case "html":
                shadowEmailContent.innerHTML = parsedEmail.html;
                break;
            case "text":
                shadowEmailContent.innerHTML = parsedEmail.text;
                break;
            case "raw":
                shadowEmailContent = document.createElement("pre");
                if (selectedEmailId) {
                    const email = emails.get(selectedEmailId);
                    if (email) {
                        shadowEmailContent.innerHTML = escapeHTML(
                            email.content,
                        );
                    }
                }
                break;
            case "headers":
                shadowEmailContent.innerHTML = "";
                break;
        }
        shadow.appendChild(shadowEmailContent);
    }

    $effect(() => {
        if (selectedEmailId !== null) {
            const email = emails.get(selectedEmailId);
            if (email) {
                parseEmail(email.content).then((result) => {
                    // console.log(result);
                    parsedEmail = result;
                    updateShownContent();
                });
            }
        }
    });

    onMount(async () => {
        const user = localStorage.getItem("user");
        if (!user) {
            window.location.href = `${base}/`;
        } else {
            const decodedUser = JSON.parse(user!);
            const response = await fetch(
                `${endpoint}/emails?username=${decodedUser.username}&password=${decodedUser.password}&page=${page}`,
            );
            if (response.ok) {
                const data = await response.json();
                emails = new Map(data.map((email: any) => [email.id, email]));
            }
        }
    });
</script>

<div>
    <!-- Off-canvas menu for mobile, show/hide based on off-canvas menu state. -->
    <div
        class="relative z-50 lg:hidden"
        role="dialog"
        aria-modal="true"
        class:hidden={!isMobileMenuOpen}
    >
        <!--
        Off-canvas menu backdrop, show/hide based on off-canvas menu state.

        Entering: "transition-opacity ease-linear duration-300"
          From: "opacity-0"
          To: "opacity-100"
        Leaving: "transition-opacity ease-linear duration-300"
          From: "opacity-100"
          To: "opacity-0"
      -->
        <div class="fixed inset-0 bg-gray-900/80"></div>

        <div class="fixed inset-0 flex">
            <!--
          Off-canvas menu, show/hide based on off-canvas menu state.

          Entering: "transition ease-in-out duration-300 transform"
            From: "-translate-x-full"
            To: "translate-x-0"
          Leaving: "transition ease-in-out duration-300 transform"
            From: "translate-x-0"
            To: "-translate-x-full"
        -->
            <div class="relative mr-16 flex w-full max-w-xs flex-1">
                <!--
            Close button, show/hide based on off-canvas menu state.

            Entering: "ease-in-out duration-300"
              From: "opacity-0"
              To: "opacity-100"
            Leaving: "ease-in-out duration-300"
              From: "opacity-100"
              To: "opacity-0"
          -->
                <div
                    class="absolute left-full top-0 flex w-16 justify-center pt-5"
                >
                    <button
                        type="button"
                        class="-m-2.5 p-2.5"
                        onclick={toggleMobileMenu}
                    >
                        <span class="sr-only">Close sidebar</span>
                        <svg
                            class="h-6 w-6 text-white"
                            fill="none"
                            viewBox="0 0 24 24"
                            stroke-width="1.5"
                            stroke="currentColor"
                            aria-hidden="true"
                        >
                            <path
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                d="M6 18L18 6M6 6l12 12"
                            />
                        </svg>
                    </button>
                </div>

                <div
                    class="flex grow flex-col gap-y-5 overflow-y-auto bg-gray-900 px-6 pb-2 ring-1 ring-white/10"
                >
                    <div class="flex h-16 shrink-0 items-center">
                        <img
                            class="h-8 w-auto"
                            src="{base}/logo-white.svg"
                            alt="Your Company"
                        />
                    </div>
                    <nav class="flex flex-1 flex-col">
                        <ul role="list" class="-mx-2 flex-1 space-y-1">
                            <li>
                                <!-- Current: "bg-gray-800 text-white", Default: "text-gray-400 hover:text-white hover:bg-gray-800" -->
                                <a
                                    href="#"
                                    class="bg-gray-800 text-white group flex gap-x-3 rounded-md p-2 text-sm leading-6 font-semibold"
                                >
                                    <svg
                                        class="h-6 w-6 shrink-0"
                                        fill="none"
                                        viewBox="0 0 24 24"
                                        stroke-width="1.5"
                                        stroke="currentColor"
                                        aria-hidden="true"
                                    >
                                        <path
                                            stroke-linecap="round"
                                            stroke-linejoin="round"
                                            d="M2.25 12l8.954-8.955c.44-.439 1.152-.439 1.591 0L21.75 12M4.5 9.75v10.125c0 .621.504 1.125 1.125 1.125H9.75v-4.875c0-.621.504-1.125 1.125-1.125h2.25c.621 0 1.125.504 1.125 1.125V21h4.125c.621 0 1.125-.504 1.125-1.125V9.75M8.25 21h8.25"
                                        />
                                    </svg>
                                    Dashboard
                                </a>
                            </li>
                        </ul>
                    </nav>
                </div>
            </div>
        </div>
    </div>

    <!-- Static sidebar for desktop -->
    <div
        class="hidden lg:fixed lg:inset-y-0 lg:left-0 lg:z-50 lg:block lg:w-20 lg:overflow-y-auto lg:bg-gray-900 lg:pb-4"
    >
        <div class="flex h-16 shrink-0 items-center justify-center">
            <img
                class="h-8 w-auto"
                src="{base}/logo-white.svg"
                alt="Your Company"
            />
        </div>
        <nav class="mt-8">
            <ul role="list" class="flex flex-col items-center space-y-1">
                <li>
                    <!-- Current: "bg-gray-800 text-white", Default: "text-gray-400 hover:text-white hover:bg-gray-800" -->
                    <a
                        href="#"
                        class="bg-gray-800 text-white group flex gap-x-3 rounded-md p-3 text-sm leading-6 font-semibold"
                    >
                        <svg
                            class="h-6 w-6 shrink-0"
                            fill="none"
                            viewBox="0 0 24 24"
                            stroke-width="1.5"
                            stroke="currentColor"
                            aria-hidden="true"
                        >
                            <path
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                d="M2.25 12l8.954-8.955c.44-.439 1.152-.439 1.591 0L21.75 12M4.5 9.75v10.125c0 .621.504 1.125 1.125 1.125H9.75v-4.875c0-.621.504-1.125 1.125-1.125h2.25c.621 0 1.125.504 1.125 1.125V21h4.125c.621 0 1.125-.504 1.125-1.125V9.75M8.25 21h8.25"
                            />
                        </svg>
                        <span class="sr-only">Dashboard</span>
                    </a>
                </li>
            </ul>
        </nav>
    </div>

    <div class="lg:pl-20">
        <div
            class="sticky top-0 z-40 flex h-16 shrink-0 items-center gap-x-4 border-b border-gray-200 bg-white px-4 shadow-sm sm:gap-x-6 sm:px-6 lg:px-8"
        >
            <button
                type="button"
                class="-m-2.5 p-2.5 text-gray-700 lg:hidden"
                onclick={toggleMobileMenu}
            >
                <span class="sr-only">Open sidebar</span>
                <svg
                    class="h-6 w-6"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke-width="1.5"
                    stroke="currentColor"
                    aria-hidden="true"
                >
                    <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        d="M3.75 6.75h16.5M3.75 12h16.5m-16.5 5.25h16.5"
                    />
                </svg>
            </button>

            <!-- Separator -->
            <div
                class="h-6 w-px bg-gray-900/10 lg:hidden"
                aria-hidden="true"
            ></div>

            <div class="flex flex-1 gap-x-4 self-stretch lg:gap-x-6">
                <form class="relative flex flex-1" action="#" method="GET">
                    <label for="search-field" class="sr-only">Search</label>
                    <svg
                        class="pointer-events-none absolute inset-y-0 left-0 h-full w-5 text-gray-400"
                        viewBox="0 0 20 20"
                        fill="currentColor"
                        aria-hidden="true"
                    >
                        <path
                            fill-rule="evenodd"
                            d="M9 3.5a5.5 5.5 0 100 11 5.5 5.5 0 000-11zM2 9a7 7 0 1112.452 4.391l3.328 3.329a.75.75 0 11-1.06 1.06l-3.329-3.328A7 7 0 012 9z"
                            clip-rule="evenodd"
                        />
                    </svg>
                    <input
                        id="search-field"
                        class="block h-full w-full border-0 py-0 pl-8 pr-0 text-gray-900 placeholder:text-gray-400 focus:ring-0 sm:text-sm"
                        placeholder="Search..."
                        type="search"
                        name="search"
                    />
                </form>
                <div class="flex items-center gap-x-4 lg:gap-x-6">
                    <button
                        type="button"
                        class="-m-2.5 p-2.5 text-gray-400 hover:text-gray-500"
                    >
                        <span class="sr-only">View notifications</span>
                        <svg
                            class="h-6 w-6"
                            fill="none"
                            viewBox="0 0 24 24"
                            stroke-width="1.5"
                            stroke="currentColor"
                            aria-hidden="true"
                        >
                            <path
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                d="M14.857 17.082a23.848 23.848 0 005.454-1.31A8.967 8.967 0 0118 9.75v-.7V9A6 6 0 006 9v.75a8.967 8.967 0 01-2.312 6.022c1.733.64 3.56 1.085 5.455 1.31m5.714 0a24.255 24.255 0 01-5.714 0m5.714 0a3 3 0 11-5.714 0"
                            />
                        </svg>
                    </button>

                    <!-- Separator -->
                    <div
                        class="hidden lg:block lg:h-6 lg:w-px lg:bg-gray-900/10"
                        aria-hidden="true"
                    ></div>

                    <!-- Profile dropdown -->
                    <div class="relative">
                        <button
                            type="button"
                            class="-m-1.5 flex items-center p-1.5"
                            id="user-menu-button"
                            aria-expanded="false"
                            aria-haspopup="true"
                            onclick={toggleUserMenu}
                        >
                            <span class="sr-only">Open user menu</span>
                            <img
                                class="h-8 w-8 rounded-full bg-gray-50"
                                src="https://images.unsplash.com/photo-1472099645785-5658abf4ff4e?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=facearea&facepad=2&w=256&h=256&q=80"
                                alt=""
                            />
                            <span class="hidden lg:flex lg:items-center">
                                <span
                                    class="ml-4 text-sm font-semibold leading-6 text-gray-900"
                                    aria-hidden="true">Tom Cook</span
                                >
                                <svg
                                    class="ml-2 h-5 w-5 text-gray-400"
                                    viewBox="0 0 20 20"
                                    fill="currentColor"
                                    aria-hidden="true"
                                >
                                    <path
                                        fill-rule="evenodd"
                                        d="M5.23 7.21a.75.75 0 011.06.02L10 11.168l3.71-3.938a.75.75 0 111.08 1.04l-4.25 4.5a.75.75 0 01-1.08 0l-4.25-4.5a.75.75 0 01.02-1.06z"
                                        clip-rule="evenodd"
                                    />
                                </svg>
                            </span>
                        </button>

                        <!--
                Dropdown menu, show/hide based on menu state.

                Entering: "transition ease-out duration-100"
                  From: "transform opacity-0 scale-95"
                  To: "transform opacity-100 scale-100"
                Leaving: "transition ease-in duration-75"
                  From: "transform opacity-100 scale-100"
                  To: "transform opacity-0 scale-95"
              -->
                        <div
                            class="absolute right-0 z-10 mt-2.5 w-32 origin-top-right rounded-md bg-white py-2 shadow-lg ring-1 ring-gray-900/5 focus:outline-none"
                            role="menu"
                            aria-orientation="vertical"
                            aria-labelledby="user-menu-button"
                            tabindex="-1"
                            class:hidden={!isUserMenuOpen}
                        >
                            <!-- Active: "bg-gray-50", Not Active: "" -->
                            <!-- <a
                                href="#"
                                class="block px-3 py-1 text-sm leading-6 text-gray-900"
                                role="menuitem"
                                tabindex="-1"
                                id="user-menu-item-0">Your profile</a
                            > -->
                            <a
                                href="#"
                                onclick={signOut}
                                class="block px-3 py-1 text-sm leading-6 text-gray-900"
                                role="menuitem"
                                tabindex="-1"
                                id="user-menu-item-1">Sign out</a
                            >
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <main class="pl-96">
            <div class="px-4 py-10 sm:px-6 lg:px-8 lg:py-6">
                <!-- Main area -->
                {#if selectedEmailId !== null}
                    <!-- <div class="flex flex-col gap-y-4">
                        <div class="bg-white rounded-lg shadow-sm p-6">
                            <h2 class="text-lg font-semibold text-gray-900">
                                {emails.get(selectedEmailId).subject}
                            </h2>
                            <p class="text-sm text-gray-500">
                                {selectedEmailContent}
                            </p>
                        </div>
                    </div> -->
                    <div class="border-b border-gray-200 pb-5">
                        <div
                            class="sm:flex sm:items-baseline sm:justify-between"
                        >
                            <div class="sm:w-0 sm:flex-1">
                                <h1
                                    id="message-heading"
                                    class="text-base font-semibold leading-6 text-gray-900"
                                >
                                    {parsedEmail ? parsedEmail.subject : ""}
                                </h1>
                                <p class="mt-1 truncate text-sm text-gray-500">
                                    From: {parsedEmail
                                        ? formatEmails([parsedEmail.from])
                                        : ""}
                                </p>
                                <p class="mt-1 truncate text-sm text-gray-500">
                                    To: {parsedEmail
                                        ? formatEmails(parsedEmail.to)
                                        : ""}
                                </p>
                                {#if parsedEmail?.cc}
                                    <p
                                        class="mt-1 truncate text-sm text-gray-500"
                                    >
                                        Cc: {parsedEmail
                                            ? formatEmails(parsedEmail.cc)
                                            : ""}
                                    </p>
                                {/if}
                            </div>

                            <div
                                class="mt-4 flex items-center justify-between sm:ml-6 sm:mt-0 sm:flex-shrink-0 sm:justify-start"
                            >
                                <span
                                    class="inline-flex items-center rounded-full bg-blue-50 px-2 py-1 text-xs font-medium text-blue-700 ring-1 ring-inset ring-blue-600/20"
                                    >{formatDate(
                                        emails.get(selectedEmailId).timestamp,
                                        true,
                                    )}</span
                                >
                                <div
                                    class="relative ml-3 inline-block text-left"
                                >
                                    <div>
                                        <button
                                            type="button"
                                            class="-my-2 flex items-center rounded-full bg-white p-2 text-gray-400 hover:text-gray-600 focus:outline-none focus:ring-2 focus:ring-indigo-500"
                                            id="menu-0-button"
                                            aria-expanded="false"
                                            aria-haspopup="true"
                                            onclick={toggleSelectedEmailMenu}
                                        >
                                            <span class="sr-only"
                                                >Open options</span
                                            >
                                            <svg
                                                class="h-5 w-5"
                                                viewBox="0 0 20 20"
                                                fill="currentColor"
                                                aria-hidden="true"
                                            >
                                                <path
                                                    d="M10 3a1.5 1.5 0 110 3 1.5 1.5 0 010-3zM10 8.5a1.5 1.5 0 110 3 1.5 1.5 0 010-3zM11.5 15.5a1.5 1.5 0 10-3 0 1.5 1.5 0 003 0z"
                                                />
                                            </svg>
                                        </button>
                                    </div>

                                    <!--
                                Dropdown menu, show/hide based on menu state.

                                Entering: "transition ease-out duration-100"
                                  From: "transform opacity-0 scale-95"
                                  To: "transform opacity-100 scale-100"
                                Leaving: "transition ease-in duration-75"
                                  From: "transform opacity-100 scale-100"
                                  To: "transform opacity-0 scale-95"
                              -->
                                    <div
                                        class="absolute right-0 z-10 mt-2 w-56 origin-top-right rounded-md bg-white shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none"
                                        role="menu"
                                        aria-orientation="vertical"
                                        aria-labelledby="menu-0-button"
                                        tabindex="-1"
                                        class:hidden={!isSelectedEmailMenuOpen}
                                    >
                                        <div class="py-1" role="none">
                                            <!-- Active: "bg-gray-100 text-gray-900", Not Active: "text-gray-700" -->
                                            <a
                                                href="#"
                                                class="text-gray-700 flex justify-between px-4 py-2 text-sm"
                                                role="menuitem"
                                                tabindex="-1"
                                                id="menu-0-item-0"
                                            >
                                                <span>Edit</span>
                                            </a>
                                            <a
                                                href="#"
                                                class="text-gray-700 flex justify-between px-4 py-2 text-sm"
                                                role="menuitem"
                                                tabindex="-1"
                                                id="menu-0-item-1"
                                            >
                                                <span>Duplicate</span>
                                            </a>
                                            <button
                                                type="button"
                                                class="text-gray-700 flex w-full justify-between px-4 py-2 text-sm"
                                                role="menuitem"
                                                tabindex="-1"
                                                id="menu-0-item-2"
                                            >
                                                <span>Archive</span>
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- tabs main content type -->
                    <div
                        class="mt-4 flex space-x-4 flex-col sm:flex-row justify-between"
                    >
                        <div>
                            <nav aria-label="Tabs">
                                <button
                                    onclick={() => setEmailViewMode("html")}
                                    class="rounded-md px-3 py-2 text-sm font-medium {selectedEmailViewMode ===
                                    'html'
                                        ? 'bg-indigo-100 text-indigo-700'
                                        : 'text-gray-500 hover:text-gray-700'}"
                                >
                                    <div class="flex items-center gap-1">
                                        <svg
                                            xmlns="http://www.w3.org/2000/svg"
                                            fill="none"
                                            viewBox="0 0 24 24"
                                            stroke-width="1.5"
                                            stroke="currentColor"
                                            class="size-6"
                                        >
                                            <path
                                                stroke-linecap="round"
                                                stroke-linejoin="round"
                                                d="M14.25 9.75 16.5 12l-2.25 2.25m-4.5 0L7.5 12l2.25-2.25M6 20.25h12A2.25 2.25 0 0 0 20.25 18V6A2.25 2.25 0 0 0 18 3.75H6A2.25 2.25 0 0 0 3.75 6v12A2.25 2.25 0 0 0 6 20.25Z"
                                            />
                                        </svg>
                                        Html
                                    </div>
                                </button>
                                <button
                                    onclick={() => setEmailViewMode("text")}
                                    class="rounded-md px-3 py-2 text-sm font-medium {selectedEmailViewMode ===
                                    'text'
                                        ? 'bg-indigo-100 text-indigo-700'
                                        : 'text-gray-500 hover:text-gray-700'} {parsedEmail &&
                                    parsedEmail.text
                                        ? ''
                                        : 'cursor-not-allowed opacity-50'}"
                                >
                                    <div class="flex items-center gap-1">
                                        <svg
                                            xmlns="http://www.w3.org/2000/svg"
                                            fill="none"
                                            viewBox="0 0 24 24"
                                            stroke-width="1.5"
                                            stroke="currentColor"
                                            class="size-6"
                                        >
                                            <path
                                                stroke-linecap="round"
                                                stroke-linejoin="round"
                                                d="M3.75 6.75h16.5M3.75 12h16.5m-16.5 5.25H12"
                                            />
                                        </svg>
                                        Text
                                    </div>
                                </button>
                                <button
                                    onclick={() => setEmailViewMode("raw")}
                                    class="rounded-md px-3 py-2 text-sm font-medium {selectedEmailViewMode ===
                                    'raw'
                                        ? 'bg-indigo-100 text-indigo-700'
                                        : 'text-gray-500 hover:text-gray-700'}"
                                    aria-current="page"
                                >
                                    <div class="flex items-center gap-1">
                                        <svg
                                            xmlns="http://www.w3.org/2000/svg"
                                            fill="none"
                                            viewBox="0 0 24 24"
                                            stroke-width="1.5"
                                            stroke="currentColor"
                                            class="size-6"
                                        >
                                            <path
                                                stroke-linecap="round"
                                                stroke-linejoin="round"
                                                d="m6.75 7.5 3 2.25-3 2.25m4.5 0h3m-9 8.25h13.5A2.25 2.25 0 0 0 21 18V6a2.25 2.25 0 0 0-2.25-2.25H5.25A2.25 2.25 0 0 0 3 6v12a2.25 2.25 0 0 0 2.25 2.25Z"
                                            />
                                        </svg>
                                        Raw
                                    </div>
                                </button>
                                <button
                                    onclick={() => setEmailViewMode("headers")}
                                    class="rounded-md px-3 py-2 text-sm font-medium {selectedEmailViewMode ===
                                    'headers'
                                        ? 'bg-indigo-100 text-indigo-700'
                                        : 'text-gray-500 hover:text-gray-700'}"
                                >
                                    <div class="flex items-center gap-1">
                                        <svg
                                            xmlns="http://www.w3.org/2000/svg"
                                            fill="none"
                                            viewBox="0 0 24 24"
                                            stroke-width="1.5"
                                            stroke="currentColor"
                                            class="size-6"
                                        >
                                            <path
                                                stroke-linecap="round"
                                                stroke-linejoin="round"
                                                d="M9.568 3H5.25A2.25 2.25 0 0 0 3 5.25v4.318c0 .597.237 1.17.659 1.591l9.581 9.581c.699.699 1.78.872 2.607.33a18.095 18.095 0 0 0 5.223-5.223c.542-.827.369-1.908-.33-2.607L11.16 3.66A2.25 2.25 0 0 0 9.568 3Z"
                                            />
                                            <path
                                                stroke-linecap="round"
                                                stroke-linejoin="round"
                                                d="M6 6h.008v.008H6V6Z"
                                            />
                                        </svg>
                                        Headers
                                    </div>
                                </button>
                            </nav>
                        </div>
                        <div class="flex justify-center">
                            <nav class="flex w-fit" aria-label="Tabs">
                                <button
                                    onclick={() => setPreviewLayout("desktop")}
                                    class="rounded-md px-3 py-2 text-sm font-medium {selectedPreviewLayout ===
                                    'desktop'
                                        ? 'bg-indigo-100 text-indigo-700'
                                        : 'text-gray-500 hover:text-gray-700'}"
                                >
                                    <div class="flex items-center gap-1">
                                        <svg
                                            xmlns="http://www.w3.org/2000/svg"
                                            fill="none"
                                            viewBox="0 0 24 24"
                                            stroke-width="1.5"
                                            stroke="currentColor"
                                            class="-ml-0.5 mr-2 h-5 w-5 {selectedPreviewLayout ===
                                            'desktop'
                                                ? 'text-indigo-500'
                                                : 'text-gray-400 group-hover:text-gray-500'}"
                                        >
                                            <path
                                                stroke-linecap="round"
                                                stroke-linejoin="round"
                                                d="M9 17.25v1.007a3 3 0 0 1-.879 2.122L7.5 21h9l-.621-.621A3 3 0 0 1 15 18.257V17.25m6-12V15a2.25 2.25 0 0 1-2.25 2.25H5.25A2.25 2.25 0 0 1 3 15V5.25m18 0A2.25 2.25 0 0 0 18.75 3H5.25A2.25 2.25 0 0 0 3 5.25m18 0V12a2.25 2.25 0 0 1-2.25 2.25H5.25A2.25 2.25 0 0 1 3 12V5.25"
                                            />
                                        </svg>
                                        Desktop
                                    </div>
                                </button>
                                <button
                                    onclick={() => setPreviewLayout("tablet")}
                                    class="rounded-md px-3 py-2 text-sm font-medium {selectedPreviewLayout ===
                                    'tablet'
                                        ? 'bg-indigo-100 text-indigo-700'
                                        : 'text-gray-500 hover:text-gray-700'}"
                                >
                                    <div class="flex items-center gap-1">
                                        <svg
                                            xmlns="http://www.w3.org/2000/svg"
                                            fill="none"
                                            viewBox="0 0 24 24"
                                            stroke-width="1.5"
                                            stroke="currentColor"
                                            class="-ml-0.5 mr-2 h-5 w-5 {selectedPreviewLayout ===
                                            'tablet'
                                                ? 'text-indigo-500'
                                                : 'text-gray-400 group-hover:text-gray-500'}"
                                        >
                                            <path
                                                stroke-linecap="round"
                                                stroke-linejoin="round"
                                                d="M10.5 19.5h3m-6.75 2.25h10.5a2.25 2.25 0 0 0 2.25-2.25v-15a2.25 2.25 0 0 0-2.25-2.25H6.75A2.25 2.25 0 0 0 4.5 4.5v15a2.25 2.25 0 0 0 2.25 2.25Z"
                                            />
                                        </svg>
                                        Tablet
                                    </div>
                                </button>
                                <button
                                    onclick={() => setPreviewLayout("mobile")}
                                    class="rounded-md px-3 py-2 text-sm font-medium {selectedPreviewLayout ===
                                    'mobile'
                                        ? 'bg-indigo-100 text-indigo-700'
                                        : 'text-gray-500 hover:text-gray-700'}"
                                >
                                    <div class="flex items-center gap-1">
                                        <svg
                                            xmlns="http://www.w3.org/2000/svg"
                                            fill="none"
                                            viewBox="0 0 24 24"
                                            stroke-width="1.5"
                                            stroke="currentColor"
                                            class="-ml-0.5 mr-2 h-5 w-5 {selectedPreviewLayout ===
                                            'mobile'
                                                ? 'text-indigo-500'
                                                : 'text-gray-400 group-hover:text-gray-500'}"
                                        >
                                            <path
                                                stroke-linecap="round"
                                                stroke-linejoin="round"
                                                d="M10.5 19.5h3m-6.75 2.25h10.5a2.25 2.25 0 0 0 2.25-2.25v-15a2.25 2.25 0 0 0-2.25-2.25H6.75A2.25 2.25 0 0 0 4.5 4.5v15a2.25 2.25 0 0 0 2.25 2.25Z"
                                            />
                                        </svg>
                                        Mobile
                                    </div>
                                </button>
                            </nav>
                        </div>
                    </div>
                    {#if parsedEmail && parsedEmail.attachments && parsedEmail.attachments.length > 0}
                        <ul role="list" class="flex mt-4 gap-4 flex-wrap">
                            {#each parsedEmail.attachments as attachment}
                                <li
                                    class="overflow-hidden rounded-xl border border-gray-200"
                                >
                                    <button
                                        class="flex items-center gap-x-2 border-b border-gray-900/5 bg-gray-50 p-2 cursor-pointer"
                                        onclick={() =>
                                            downloadAttachment(attachment)}
                                    >
                                        {#if attachment.mimeType && attachment.mimeType.includes("image")}
                                            <svg
                                                xmlns="http://www.w3.org/2000/svg"
                                                fill="none"
                                                viewBox="0 0 24 24"
                                                stroke-width="1.5"
                                                stroke="currentColor"
                                                class="size-6 flex-none text-gray-600"
                                            >
                                                <path
                                                    stroke-linecap="round"
                                                    stroke-linejoin="round"
                                                    d="m2.25 15.75 5.159-5.159a2.25 2.25 0 0 1 3.182 0l5.159 5.159m-1.5-1.5 1.409-1.409a2.25 2.25 0 0 1 3.182 0l2.909 2.909m-18 3.75h16.5a1.5 1.5 0 0 0 1.5-1.5V6a1.5 1.5 0 0 0-1.5-1.5H3.75A1.5 1.5 0 0 0 2.25 6v12a1.5 1.5 0 0 0 1.5 1.5Zm10.5-11.25h.008v.008h-.008V8.25Zm.375 0a.375.375 0 1 1-.75 0 .375.375 0 0 1 .75 0Z"
                                                />
                                            </svg>
                                        {:else if attachment.mimeType && attachment.mimeType.includes("video")}
                                            <svg
                                                xmlns="http://www.w3.org/2000/svg"
                                                fill="none"
                                                viewBox="0 0 24 24"
                                                stroke-width="1.5"
                                                stroke="currentColor"
                                                class="size-6 flex-none text-gray-600"
                                            >
                                                <path
                                                    stroke-linecap="round"
                                                    stroke-linejoin="round"
                                                    d="M3.375 19.5h17.25m-17.25 0a1.125 1.125 0 0 1-1.125-1.125M3.375 19.5h1.5C5.496 19.5 6 18.996 6 18.375m-3.75 0V5.625m0 12.75v-1.5c0-.621.504-1.125 1.125-1.125m18.375 2.625V5.625m0 12.75c0 .621-.504 1.125-1.125 1.125m1.125-1.125v-1.5c0-.621-.504-1.125-1.125-1.125m0 3.75h-1.5A1.125 1.125 0 0 1 18 18.375M20.625 4.5H3.375m17.25 0c.621 0 1.125.504 1.125 1.125M20.625 4.5h-1.5C18.504 4.5 18 5.004 18 5.625m3.75 0v1.5c0 .621-.504 1.125-1.125 1.125M3.375 4.5c-.621 0-1.125.504-1.125 1.125M3.375 4.5h1.5C5.496 4.5 6 5.004 6 5.625m-3.75 0v1.5c0 .621.504 1.125 1.125 1.125m0 0h1.5m-1.5 0c-.621 0-1.125.504-1.125 1.125v1.5c0 .621.504 1.125 1.125 1.125m1.5-3.75C5.496 8.25 6 7.746 6 7.125v-1.5M4.875 8.25C5.496 8.25 6 8.754 6 9.375v1.5m0-5.25v5.25m0-5.25C6 5.004 6.504 4.5 7.125 4.5h9.75c.621 0 1.125.504 1.125 1.125m1.125 2.625h1.5m-1.5 0A1.125 1.125 0 0 1 18 7.125v-1.5m1.125 2.625c-.621 0-1.125.504-1.125 1.125v1.5m2.625-2.625c.621 0 1.125.504 1.125 1.125v1.5c0 .621-.504 1.125-1.125 1.125M18 5.625v5.25M7.125 12h9.75m-9.75 0A1.125 1.125 0 0 1 6 10.875M7.125 12C6.504 12 6 12.504 6 13.125m0-2.25C6 11.496 5.496 12 4.875 12M18 10.875c0 .621-.504 1.125-1.125 1.125M18 10.875c0 .621.504 1.125 1.125 1.125m-2.25 0c.621 0 1.125.504 1.125 1.125m-12 5.25v-5.25m0 5.25c0 .621.504 1.125 1.125 1.125h9.75c.621 0 1.125-.504 1.125-1.125m-12 0v-1.5c0-.621-.504-1.125-1.125-1.125M18 18.375v-5.25m0 5.25v-1.5c0-.621.504-1.125 1.125-1.125M18 13.125v1.5c0 .621.504 1.125 1.125 1.125M18 13.125c0-.621.504-1.125 1.125-1.125M6 13.125v1.5c0 .621-.504 1.125-1.125 1.125M6 13.125C6 12.504 5.496 12 4.875 12m-1.5 0h1.5m-1.5 0c-.621 0-1.125.504-1.125 1.125v1.5c0 .621.504 1.125 1.125 1.125M19.125 12h1.5m0 0c.621 0 1.125.504 1.125 1.125v1.5c0 .621-.504 1.125-1.125 1.125m-17.25 0h1.5m14.25 0h1.5"
                                                />
                                            </svg>
                                        {:else}
                                            <svg
                                                xmlns="http://www.w3.org/2000/svg"
                                                fill="none"
                                                viewBox="0 0 24 24"
                                                stroke-width="1.5"
                                                stroke="currentColor"
                                                class="size-6 flex-none text-gray-600"
                                            >
                                                <path
                                                    stroke-linecap="round"
                                                    stroke-linejoin="round"
                                                    d="M19.5 14.25v-2.625a3.375 3.375 0 0 0-3.375-3.375h-1.5A1.125 1.125 0 0 1 13.5 7.125v-1.5a3.375 3.375 0 0 0-3.375-3.375H8.25m2.25 0H5.625c-.621 0-1.125.504-1.125 1.125v17.25c0 .621.504 1.125 1.125 1.125h12.75c.621 0 1.125-.504 1.125-1.125V11.25a9 9 0 0 0-9-9Z"
                                                />
                                            </svg>
                                        {/if}
                                        <div
                                            class="text-sm font-medium leading-6 text-gray-900"
                                        >
                                            {attachment.filename}
                                        </div>
                                    </button>
                                </li>
                            {/each}
                        </ul>
                    {/if}
                    <div
                        class="mt-4 border-2 border-gray-200 rounded-lg {selectedPreviewLayout} overflow-auto mx-auto {selectedEmailViewMode ===
                        'headers'
                            ? 'hidden'
                            : ''}"
                        id="shadow-email"
                    ></div>
                    {#if selectedEmailViewMode === "headers"}
                        <div class="mt-4">
                            <div class="mt-6 border-t border-gray-100">
                                <dl class="divide-y divide-gray-100">
                                    {#each parsedEmail.headers as element, i}
                                        <div
                                            class="px-4 py-6 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-3 {i %
                                                2 ===
                                            0
                                                ? 'bg-gray-50'
                                                : 'bg-white'}"
                                        >
                                            <dt
                                                class="text-sm font-medium leading-6 text-gray-900"
                                            >
                                                {element.key}
                                            </dt>
                                            <dd
                                                class="mt-1 text-sm leading-6 text-gray-700 sm:col-span-2 sm:mt-0"
                                            >
                                                {element.value}
                                            </dd>
                                        </div>
                                    {/each}
                                </dl>
                            </div>
                        </div>
                    {/if}
                {/if}
            </div>
        </main>
    </div>

    <aside
        class="fixed bottom-0 left-20 top-16 xxl:hidden w-96 overflow-y-auto border-r border-gray-200 xl:block"
    >
        <!-- Secondary column (hidden on smaller screens) -->
        <div class="flex flex-col gap-y-4">
            <ul role="list" class="divide-y divide-gray-100">
                {#each Array.from(emails.values()) as email}
                    <li
                        class="flex justify-between gap-x-6 py-5 hover:opacity-100 px-4 sm:px-6 lg:px-8 {selectedEmailId ===
                        email.id
                            ? 'bg-gray-200 opacity-100'
                            : 'opacity-80'}"
                    >
                        <button
                            class="flex min-w-0 gap-x-4 cursor-pointer"
                            onclick={() => (selectedEmailId = email.id)}
                        >
                            <div class="flex flex-col items-center">
                                <img
                                    class="size-12 flex-none rounded-full bg-gray-50"
                                    src="https://ui-avatars.com/api/?name={email.sender}"
                                    alt=""
                                />
                                <p
                                    class="mt-1 text-wrap text-xs/5 text-gray-900"
                                >
                                    {formatDate(email.timestamp)}
                                </p>
                            </div>
                            <div
                                class="hidden min-w-0 flex-auto sm:flex flex-col items-start"
                            >
                                <p
                                    class="text-sm/6 font-semibold text-gray-900 text-left"
                                >
                                    {email.subject}
                                </p>
                                <p
                                    class="mt-1 text-wrap text-xs/5 text-gray-500 text-left"
                                >
                                    from: {email.sender}
                                </p>
                                <p
                                    class="mt-1 text-wrap text-xs/5 text-gray-500 text-left"
                                >
                                    to: {email.recipients.split(",")}
                                </p>
                            </div>
                        </button>
                    </li>
                {/each}
            </ul>
        </div>
    </aside>
</div>

<style>
    /* smmothly transition between max widths */
    #shadow-email {
        transition: all 0.5s;
    }
    #shadow-email.mobile {
        max-width: 375px;
    }
    #shadow-email.tablet {
        max-width: 768px;
    }
    #shadow-email.desktop {
        max-width: 100%;
    }
</style>
