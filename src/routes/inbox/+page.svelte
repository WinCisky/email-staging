<script lang="ts">
    import { base } from "$app/paths";
    import { onMount } from "svelte";

    const endpoint = "http://localhost:3000";

    let isMobileMenuOpen = $state(false);
    let isUserMenuOpen = $state(false);
    let emails = $state(new Map<number, any>());
    let page = $state(1);
    let selectedEmailId: number | null = $state(null);
    let selectedEmailContent: string | null = $state(null);

    function toggleMobileMenu() {
        isMobileMenuOpen = !isMobileMenuOpen;
    }

    function toggleUserMenu() {
        isUserMenuOpen = !isUserMenuOpen;
    }

    function signOut() {
        localStorage.removeItem("user");
        window.location.href = `${base}/`;
    }

    function parseEmail(emailContent: string) {
        // Dividi l'email in intestazioni e corpo
        const [headers, ...bodyParts] = emailContent.split("\r\n\r\n");
        const body = bodyParts.join("\r\n\r\n");

        // Dividi le intestazioni in singole righe
        const headerLines = headers.split("\r\n");
        const headerMap: any = {};

        // Crea una mappa delle intestazioni
        headerLines.forEach((line) => {
            const [key, value] = line.split(": ");
            headerMap[key] = value;
        });

        return {
            headers: headerMap,
            body: body,
        };
    }

    function formatDate(dateString: string): string {
        console.log(dateString);
        const locale = "it-IT";
        const timeZone = "Europe/Rome";

        const date = new Date(dateString);
        const today = new Date();

        // Check if the date is today
        const isToday = date.getDate() === today.getDate() &&
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

    $effect(() => {
        if (selectedEmailId !== null) {
            const email = emails.get(selectedEmailId);
            if (email) {
                selectedEmailContent = parseEmail(email.content).body ?? null;
            } else {
                selectedEmailContent = null;
            }
        } else {
            selectedEmailContent = null;
        }
    });

    onMount(async () => {
        const user = localStorage.getItem("user");
        if (!user) {
            window.location.href = `${base}/`;
        } else {
            // const data = [{"id":1,"username":"your_username1","password":"your_password2","sender":"sender@example.com","recipients":"recipient@example.com","subject":"Hello","content":"From: Sender Name <sender@example.com>\r\nTo: recipient@example.com\r\nSubject: Hello\r\nMessage-ID: <226dcc5b-4d8a-6f44-7c45-266fc3e9f5a8@example.com>\r\nDate: Tue, 03 Dec 2024 11:39:01 +0000\r\nMIME-Version: 1.0\r\nContent-Type: multipart/alternative;\r\n boundary=\"--_NmP-1e24ea08a4e0b6fe-Part_1\"\r\n\r\n----_NmP-1e24ea08a4e0b6fe-Part_1\r\nContent-Type: text/plain; charset=utf-8\r\nContent-Transfer-Encoding: 7bit\r\n\r\nHello world?\r\n----_NmP-1e24ea08a4e0b6fe-Part_1\r\nContent-Type: text/html; charset=utf-8\r\nContent-Transfer-Encoding: 7bit\r\n\r\n<b>Hello world?</b>\r\n----_NmP-1e24ea08a4e0b6fe-Part_1--\r\n","timestamp":"2024-12-03 11:39:02"},{"id":2,"username":"your_username1","password":"your_password2","sender":"sender@example.com","recipients":"recipient@example.com","subject":"Hello","content":"From: Sender Name <sender@example.com>\r\nTo: recipient@example.com\r\nSubject: Hello\r\nMessage-ID: <842e18e7-0e8e-b526-9f46-1126b13cf46a@example.com>\r\nDate: Tue, 03 Dec 2024 11:39:05 +0000\r\nMIME-Version: 1.0\r\nContent-Type: multipart/alternative;\r\n boundary=\"--_NmP-c758aa0442799687-Part_1\"\r\n\r\n----_NmP-c758aa0442799687-Part_1\r\nContent-Type: text/plain; charset=utf-8\r\nContent-Transfer-Encoding: 7bit\r\n\r\nHello world?\r\n----_NmP-c758aa0442799687-Part_1\r\nContent-Type: text/html; charset=utf-8\r\nContent-Transfer-Encoding: 7bit\r\n\r\n<b>Hello world?</b>\r\n----_NmP-c758aa0442799687-Part_1--\r\n","timestamp":"2024-12-03 11:39:05"},{"id":3,"username":"your_username1","password":"your_password2","sender":"sender@example.com","recipients":"recipient@example.com","subject":"Hello","content":"From: Sender Name <sender@example.com>\r\nTo: recipient@example.com\r\nSubject: Hello\r\nMessage-ID: <4b244f2f-df0a-ed4d-5c7d-efad5782b022@example.com>\r\nDate: Tue, 03 Dec 2024 11:39:06 +0000\r\nMIME-Version: 1.0\r\nContent-Type: multipart/alternative;\r\n boundary=\"--_NmP-7e0f1c14217829ca-Part_1\"\r\n\r\n----_NmP-7e0f1c14217829ca-Part_1\r\nContent-Type: text/plain; charset=utf-8\r\nContent-Transfer-Encoding: 7bit\r\n\r\nHello world?\r\n----_NmP-7e0f1c14217829ca-Part_1\r\nContent-Type: text/html; charset=utf-8\r\nContent-Transfer-Encoding: 7bit\r\n\r\n<b>Hello world?</b>\r\n----_NmP-7e0f1c14217829ca-Part_1--\r\n","timestamp":"2024-12-03 11:39:06"}];
            // emails = new Map(data.map((email: any) => [email.id, email]));
            // return;
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
                    <div class="flex flex-col gap-y-4">
                        <div class="bg-white rounded-lg shadow-sm p-6">
                            <h2 class="text-lg font-semibold text-gray-900">
                                {emails.get(selectedEmailId).subject}
                            </h2>
                            <p class="text-sm text-gray-500">
                                {selectedEmailContent}
                            </p>
                        </div>
                    </div>
                {/if}
            </div>
        </main>
    </div>

    <aside
        class="fixed bottom-0 left-20 top-16 xxl:hidden w-96 overflow-y-auto border-r border-gray-200 px-4 py-6 sm:px-6 lg:px-8 xl:block"
    >
        <!-- Secondary column (hidden on smaller screens) -->
        <div class="flex flex-col gap-y-4">
            <ul role="list" class="divide-y divide-gray-100">
                {#each Array.from(emails.values()) as email}
                <li class="flex justify-between gap-x-6 py-5">
                  <button class="flex min-w-0 gap-x-4 cursor-pointer" onclick={() => selectedEmailId = email.id}>
                    <div class="flex flex-col items-center">
                        <img class="size-12 flex-none rounded-full bg-gray-50" src="https://ui-avatars.com/api/?name={email.sender}" alt="">
                        <p class="mt-1 text-wrap text-xs/5 text-gray-900">{formatDate(email.timestamp)}</p>
                    </div>
                    <div class="min-w-0 flex-auto flex flex-col items-start">
                      <p class="text-sm/6 font-semibold text-gray-900">{email.subject}</p>
                      <p class="mt-1 text-wrap text-xs/5 text-gray-500">from: {email.sender}</p>
                      <p class="mt-1 text-wrap text-xs/5 text-gray-500">to: {email.recipients.split(",")}</p>
                    </div>
                </button>
                </li>
                {/each}
            </ul>
        </div>
    </aside>
</div>
