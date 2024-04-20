<script>
    import { onMount } from 'svelte';
    let isLoggedIn = false;

    onMount(() => {
        const ARLcookie = getCookie('ARL');

        if (!ARLcookie) {
            window.location.href = '/user';
            return;
        } else {
            isLoggedIn = true;
            window.location.href = '/calendar';
            return;
        }
    });

    function getCookie(name) {
        const cookies = document.cookie.split(';');
        for (let i = 0; i < cookies.length; i++) {
            const cookie = cookies[i].trim();
            if (cookie.startsWith(name + '=')) {
                return cookie.substring(name.length + 1);
            }
        }
        return null;
    }
</script>