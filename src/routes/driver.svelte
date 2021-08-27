<script>
	import AcountAlert from "../components/AcountAlert.svelte";
	import { stores } from '@sapper/app';
	import { onMount } from 'svelte'
	import { goto } from '@sapper/app'

	const { session } = stores()

	// De la posicion actual de la persona
	let latitude, longitude
	let rutas = []
	let onRoute = ''

	// Variables globales necesarias para usar google maps
	let directionService, directionDisplay, homeMarker, map

	let estimatedTime, estimatedCost
	
	onMount(() => {
		navigator.geolocation.getCurrentPosition(async (p) => {
			latitude = p.coords.latitude;
			longitude = p.coords.longitude;

			map = new google.maps.Map(document.getElementById('map'), {
				zoom: 18,
				center: {lat: latitude, lng: longitude},
				zoomControl: true
			})
			
			// Creacion del servico de direcciones de goole maps
			directionService = new google.maps.DirectionsService()
			directionDisplay = new google.maps.DirectionsRenderer();
			directionDisplay.setMap(map)
		}, (err) => console.log(err))

		// Obtener la localizacion actual
		navigator.geolocation.watchPosition(async ({ coords: { latitude: lat, longitude: lon } }) => {
			latitude = lat
			longitude = lon
			
			// Crea el mapa
			// Agrega un marcador a la posicion actual de la persona
			homeMarker = new google.maps.Marker({
				position: {lat: latitude, lng: longitude},
				map,
				title: 'Home',
			})
			
		}, (err) => console.log(err))
	
		db.collection('routes').onSnapshot(async (snapshot) => {
			rutas = []
			snapshot.forEach(change => {
				rutas.push(change.data())
			})
		})
	})

	const handleRouteClick = async (user) => {
		db.collection('routes').doc(user)
			.update({ driver:  $session.userData.uid })

		onRoute = await db.collection('routes').doc(user).get()
		onRoute = onRoute.data()

		var request = {
			origin: {lat: latitude, lng: longitude},
			destination: onRoute.start,
			travelMode: google.maps.TravelMode.DRIVING,
			unitSystem: google.maps.UnitSystem.METRIC
		}

		directionService.route(request, (result, status) => {
			if (status == google.maps.DirectionsStatus.OK) {
				directionDisplay.setDirections(result)
			} else {
				alert(status);
				
				directionDisplay.setDirections({ routes: [] });
				map.setCenter({lat: latitude, lng: longitude});
			}
		});
	}

	const handleSignOut = () => {
        firebase.auth().signOut()
            .then(() => {
                goto('/')
            })
            .catch(err => console.error(err))
    }
</script>


<svelte:head>
	<title>That way: Driver</title>
</svelte:head>

{#if !$session.user}
	<AcountAlert></AcountAlert>
{/if}

<main>
	<div class="float-btn" on:click={handleSignOut}>SignOut</div>

	<aside>
		{#if onRoute == ''}
			{#each rutas as ruta}
				<div on:click={() => handleRouteClick(ruta.user)} class="ruta">
					<h2>Destination</h2>
					<p>{ruta.end}</p>
					<h2>Starting point</h2>
					<p>{ruta.start}</p>
					<h2>Price</h2>
					<p>{ruta.price}</p>
					<h2>Time</h2>
					<p>{ruta.time}</p>
				</div>
			{/each}
		{:else}
			<div class="ruta">
				<h2>Destination</h2>
				<p>{onRoute.end}</p>
				<h2>Starting point</h2>
				<p>{onRoute.start}</p>
				<h2>Price</h2>
				<p>{onRoute.price}</p>
				<h2>Time</h2>
				<p>{onRoute.time}</p>
			</div>
		{/if}
	</aside>
	<div id='map'></div>

</main>



<style>
	main {
		padding: 0;
		min-height: 100vh;
		background-color: #ececec;
	}

	#map {
		width: 100%;
		height: 100vh;
		z-index: 1;
	}

	aside {
		position: absolute;
		z-index: 2;
		bottom: 0;
		left: 0;
		right: 0;
		padding: 0.5rem;
		background: #ffffff;
		max-height: 40vh;
		overflow-y: scroll;
	}

	.float-btn {
		width: auto;
		z-index: 9;
		position: absolute;
		right: 10px;
		top: 10px;
		background: var(--blue-normal);
		border-radius: 1rem;
		padding: 1rem;
		box-shadow: 5px 5px 32px 4px rgba(0,0,0,0.41);
		color: #ffffff;
		cursor: pointer;	
	}

	.ruta {
		margin: 0.5rem;
		border: 2px solid gray;
		padding: 1rem;
	}
</style>