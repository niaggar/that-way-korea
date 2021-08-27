<script>
	import { onMount } from 'svelte'
    import { stores } from '@sapper/app'

	// export let segment

    const { session } = stores()


    const saveCreadentials = async () => {
        const user = await firebase.auth().currentUser
        const userInfo = await db.collection('users').doc(user.uid).get()

        $session.userData = { 
            uid: user.uid,
            email: user.email,
            ...userInfo.data(),
        }
    }

    const userManager = async (user) => {
        if (!user) {
            $session.user = false
            $session.userData = false
            return
        }
        
        setTimeout(async () => await saveCreadentials(), 3000)

        const token = await user.getIdToken()
        $session.user = token
    }


    onMount(async () => firebase.auth().onIdTokenChanged(userManager))
</script>


<slot></slot>
<!-- <Nav {segment}/> -->


<style>
    :global(body) {
        box-sizing: border-box;
        min-height: 100vh;
    }
</style>
