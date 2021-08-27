<script context="module">
	export async function preload(page, session) {
		let { user } = session
		if (user)
		    return this.redirect(302, '/')
	}
</script>

<script>
    import { goto } from '@sapper/app'

    let email = ''
    let password = ''

    const handleSubmintLogin = (e) => {
        e.preventDefault()
        
        email = email.trim()
        password = password.trim()

        firebase.auth().signInWithEmailAndPassword(email, password)
            .then((user) => goto('/'))
            .catch((err) => console.error('AUTH ERR ' + err))
    }
</script>


<div class="container">
    <div>
        <form on:submit={handleSubmintLogin}>
            <input required bind:value={email} placeholder="Email" type="email">
            <input required bind:value={password} placeholder="Password" type="password">
            <button type="submit">Login</button>
        </form>
    </div>
    <a href="auth">Back</a>
</div>


<style>
   .container * {
        display: block;
        width: 100%;
    }

    .container {
        padding: 0 1rem;
        width: 100%;
        min-height: 100vh;
        background-color: var(--dark);
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
    }

    form * {
        padding: 0.8em;
        margin: 0.5em 0;
        outline: none;
        border: none;
        font-size: 1rem;
        transition: all 0.3s;
    }

    form input {
        background: none;
        border-bottom: 4px solid var(--blue-normal);
        color: var(--blue-normal);
        margin: 1em 0;
    }

    form input::placeholder {
        color: var(--blue-normal);
        opacity: 0.7;
    }

    form input:active,
    form input:focus {
        box-shadow: 0 0 0 2.5px var(--blue-light);
    }

    form button {
        text-align: center;
        cursor: pointer;
        font-weight: 700;
        border-radius: 0.3em;
        background-color: var(--blue-light);
        color: #000000;
        margin-top: 1em;
    }

    form button:hover {
        filter: brightness(1.2);
    }

    form button:active,
    form button:focus {
        box-shadow: 0 0 0 1.5px var(--blue-light);
    }

    a {
        width: 100%;
        text-align: center;
        color: var(--blue-light);
        opacity: 0.7;
        font-weight: 700;
        margin-top: 2rem;
    }
</style>