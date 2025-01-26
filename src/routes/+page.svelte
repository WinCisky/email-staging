<script lang="ts">
    import { base } from "$app/paths";
    import { onMount } from "svelte";

    let username = "";
    let password = "";
    let usernameError = "";
    let passwordError = "";

    function validateForm() {
        usernameError = "";
        passwordError = "";

        if (username.length < 3) {
            usernameError = "Username must be at least 3 characters long";
        }

        if (password.length < 6) {
            passwordError = "Password must be at least 6 characters long";
        }

        return !usernameError && !passwordError;
    }

    function handleSubmit(event: Event) {
        event.preventDefault();
        if (validateForm()) {
            localStorage.setItem(
                "user",
                JSON.stringify({ username, password }),
            );
            window.location.href = `${base}/inbox`;
        }
    }

    onMount(() => {
        const user = localStorage.getItem("user");
        if (user) {
            window.location.href = `${base}/inbox`;
        }
    });
</script>

<div
    class="flex flex-col min-h-full items-center justify-center px-4 py-12 sm:px-6 lg:px-8"
>
    <div class="w-full max-w-sm space-y-10">
        <div>
            <h2
                class="mt-10 text-center text-2xl font-bold leading-9 tracking-tight text-gray-900"
            >
                Pick your account
            </h2>
        </div>
        <form class="space-y-6" on:submit={handleSubmit}>
            <div class="relative -space-y-px rounded-md shadow-sm">
                <div
                    class="pointer-events-none absolute inset-0 z-10 rounded-md ring-1 ring-inset ring-gray-300"
                ></div>
                <div>
                    <label for="username" class="sr-only">Username</label>
                    <input
                        id="username"
                        name="username"
                        type="text"
                        bind:value={username}
                        required
                        class="relative block w-full rounded-t-md border-0 py-1.5 text-gray-900 ring-1 ring-inset ring-gray-100 placeholder:text-gray-400 focus:z-10 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6"
                        placeholder="Username"
                    />
                    {#if usernameError}
                        <p class="text-red-500 text-sm">{usernameError}</p>
                    {/if}
                </div>
                <div>
                    <label for="password" class="sr-only">Password</label>
                    <input
                        id="password"
                        name="password"
                        type="password"
                        bind:value={password}
                        required
                        class="relative block w-full rounded-b-md border-0 py-1.5 text-gray-900 ring-1 ring-inset ring-gray-100 placeholder:text-gray-400 focus:z-10 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6"
                        placeholder="Password"
                    />
                    {#if passwordError}
                        <p class="text-red-500 text-sm">{passwordError}</p>
                    {/if}
                </div>
            </div>

            <div>
                <button
                    type="submit"
                    class="flex w-full justify-center rounded-md bg-indigo-600 px-3 py-1.5 text-sm font-semibold leading-6 text-white hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
                    >Pick</button
                >
            </div>
        </form>
    </div>
</div>

<div
    class="pointer-events-none fixed inset-x-0 bottom-0 sm:flex sm:justify-center sm:px-6 sm:pb-5 lg:px-8"
>
    <div
        class="pointer-events-auto flex items-center justify-between gap-x-6 bg-amber-400 px-6 py-2.5 sm:rounded-xl sm:py-3 sm:pl-4 sm:pr-3.5"
    >
        <p class="text-sm leading-6 text-white">
            <strong class="font-semibold">Attention needed</strong><svg
                viewBox="0 0 2 2"
                class="mx-2 inline h-0.5 w-0.5 fill-current"
                aria-hidden="true"><circle cx="1" cy="1" r="1" /></svg
            >This service is only for testing purposes.
        </p>
    </div>
</div>

<div class="rounded-md bg-yellow-50 p-4">
    <div class="flex">
        <div class="flex-shrink-0">
            <svg
                class="h-5 w-5 text-yellow-400"
                viewBox="0 0 20 20"
                fill="currentColor"
                aria-hidden="true"
            >
                <path
                    fill-rule="evenodd"
                    d="M8.485 2.495c.673-1.167 2.357-1.167 3.03 0l6.28 10.875c.673 1.167-.17 2.625-1.516 2.625H3.72c-1.347 0-2.189-1.458-1.515-2.625L8.485 2.495zM10 5a.75.75 0 01.75.75v3.5a.75.75 0 01-1.5 0v-3.5A.75.75 0 0110 5zm0 9a1 1 0 100-2 1 1 0 000 2z"
                    clip-rule="evenodd"
                />
            </svg>
        </div>
        <div class="ml-3">
            <h3 class="text-sm font-medium text-yellow-800">
                Attention needed
            </h3>
            <div class="mt-2 text-sm text-yellow-700">
                <p>This service is only for testing purposes.</p>
            </div>
        </div>
    </div>
</div>
