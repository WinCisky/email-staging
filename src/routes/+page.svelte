<script lang="ts">
    import { base } from '$app/paths';
    import { onMount } from 'svelte';

    let username = '';
    let password = '';
    let usernameError = '';
    let passwordError = '';

    function validateForm() {
        usernameError = '';
        passwordError = '';

        if (username.length < 3) {
            usernameError = 'Username must be at least 3 characters long';
        }

        if (password.length < 6) {
            passwordError = 'Password must be at least 6 characters long';
        }

        return !usernameError && !passwordError;
    }

    function handleSubmit(event: Event) {
        event.preventDefault();
        if (validateForm()) {
            localStorage.setItem('user', JSON.stringify({ username, password }));
            window.location.href = `${base}/inbox`;
        }
    }

    onMount(() => {
        const user = localStorage.getItem('user');
        if (user) {
            window.location.href = `${base}/inbox`;
        }
    });
</script>

<div class="flex min-h-full items-center justify-center px-4 py-12 sm:px-6 lg:px-8">
    <div class="w-full max-w-sm space-y-10">
        <div>
            <img class="mx-auto h-10 w-auto" src="{base}/logo-black.svg" alt="Your Company">
            <h2 class="mt-10 text-center text-2xl font-bold leading-9 tracking-tight text-gray-900">Pick your account</h2>
        </div>
        <form class="space-y-6" on:submit={handleSubmit}>
            <div class="relative -space-y-px rounded-md shadow-sm">
                <div class="pointer-events-none absolute inset-0 z-10 rounded-md ring-1 ring-inset ring-gray-300"></div>
                <div>
                    <label for="username" class="sr-only">Username</label>
                    <input id="username" name="username" type="text" bind:value={username} required class="relative block w-full rounded-t-md border-0 py-1.5 text-gray-900 ring-1 ring-inset ring-gray-100 placeholder:text-gray-400 focus:z-10 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6" placeholder="Username">
                    {#if usernameError}
                        <p class="text-red-500 text-sm">{usernameError}</p>
                    {/if}
                </div>
                <div>
                    <label for="password" class="sr-only">Password</label>
                    <input id="password" name="password" type="password" bind:value={password} required class="relative block w-full rounded-b-md border-0 py-1.5 text-gray-900 ring-1 ring-inset ring-gray-100 placeholder:text-gray-400 focus:z-10 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6" placeholder="Password">
                    {#if passwordError}
                        <p class="text-red-500 text-sm">{passwordError}</p>
                    {/if}
                </div>
            </div>

            <div>
                <button type="submit" class="flex w-full justify-center rounded-md bg-indigo-600 px-3 py-1.5 text-sm font-semibold leading-6 text-white hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600">Pick</button>
            </div>
        </form>
    </div>
</div>