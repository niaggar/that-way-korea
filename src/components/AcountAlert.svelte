<script>
    let noEsVisible = false
</script>


<div class:no-visible={noEsVisible}>
    <aside>
        <p>It seems you don't have an account connected. We invite you to do so so that you can enjoy all the functionalities!</p>
        <a href="auth">Connect</a>
    </aside>
</div>


<style>
    div.no-visible {
        display: none;
    }

    div {
        position: fixed;
        z-index: 9999;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-color: #000000a2;
        display: flex;
        align-items: center;
        justify-content: center;
        padding: 0 1em;
    }

    aside {
        width: 100%;
        background-color: #ffffff;
        border-radius: 0.3rem;
        padding: 1em;
        text-align: center;
    }

    aside a {
        display: block;
        padding: 1em;
        border-radius: 0.3rem;
        background-color: var(--blue-dark);
        text-align: center;
        margin: 1em 0 0.5em 0;
        color: #ffffff;
        text-decoration: none;
    }
</style>
