<script>
	import AcountAlert from "../components/AcountAlert.svelte";
	import { stores } from '@sapper/app';
	import { onMount } from 'svelte'
	import { goto } from '@sapper/app'

	const { session } = stores()

	$: console.log(homeAddress)

	// De la posicion actual de la persona
	let latitude, longitude
	
	// Verificar si se quiere dar la direccion en lat y lng
	let homeInCord = false
	let destinationInCord = false

	// Verificar estado actual de la app
	let awaitStartRoute = false
	let awaitDriver = false
	let routeStarted = false
	
	// Direccion de la ruta
	let homeAddress = ''
	let destinationAddress = ''
	let direction

	// Variables globales necesarias para usar google maps
	let directionService, directionDisplay, homeMarker, map

	let estimatedTime, estimatedCost
	
	onMount(() => {

		setTimeout(() => {
			console.log($session.userData)
			if ($session.userData.type === 'driver')
				goto('/driver')
		}, 4000)

		// Obtener la localizacion actual
		navigator.geolocation.getCurrentPosition(async (p) => {
			latitude = p.coords.latitude;
			longitude = p.coords.longitude;
			
			console.log('Current position', latitude, longitude)
			
			// Crea el mapa
			map = new google.maps.Map(document.getElementById('map'), {
				zoom: 18,
				center: {lat: latitude, lng: longitude},
				zoomControl: true
			})
			
			// Agrega un marcador a la posicion actual de la persona
			homeMarker = new google.maps.Marker({
				position: {lat: latitude, lng: longitude},
				map,
				title: 'Home',
			})

			// Creacion del servico de direcciones de goole maps
			directionService = new google.maps.DirectionsService()
			directionDisplay = new google.maps.DirectionsRenderer();
			directionDisplay.setMap(map)

			console.log('Home Marker', homeMarker)
			console.log('Mapa', map)

		}, (error) => alert(error));
	})

	const handleSubmitForm = async (e) => {
		e.preventDefault();

		//create request
		var request = {
			origin: homeAddress,
			destination: destinationAddress,
			travelMode: google.maps.TravelMode.DRIVING,
			unitSystem: google.maps.UnitSystem.METRIC
		}

		//pass the request to the route method
		directionService.route(request, (result, status) => {
			if (status == google.maps.DirectionsStatus.OK) {
				//Get distance and time
				// const output = document.querySelector('#output');
				estimatedTime = result.routes[0].legs[0].duration.text
				estimatedCost = result.routes[0].legs[0].distance.value / 1000
				console.log(result.routes[0].legs[0])

				awaitStartRoute = true

				direction = result.routes[0].legs[0]
				//display route
				directionDisplay.setDirections(result)
			} else {
				alert(status);
				
				//delete route from map
				awaitStartRoute = false
				directionDisplay.setDirections({ routes: [] });
				//center map in actual location
				map.setCenter({lat: latitude, lng: longitude});

				//show error message
				// output.innerHTML = "<div class='alert-danger'><i class='fas fa-exclamation-triangle'></i> Could not retrieve driving distance.</div>";
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

	const handleBack = () => {
		estimatedTime = undefined
		estimatedCost = undefined

		awaitStartRoute = false
		directionDisplay.setDirections({ routes: [] });
		map.setCenter({lat: latitude, lng: longitude});
	}

	const handleRequest = () => {
		db.collection('routes').doc($session.userData.uid)
			.set({
				user: $session.userData.uid,
				time: direction.duration.text,
				price: direction.distance.value / 1000,
				driver: 'undifined',
				completed: false,
				start: direction.start_address,
				end: direction.end_address,
			})
			.then(() => console.log('info of route sett'))
			.catch((err) => console.error(err))

		db.collection('routes').doc($session.userData.uid)
			.onSnapshot(async (doc) => {
				console.log(doc.data())

				if (doc.data().driver != 'undifined')
					routeStarted = true
			})
		
		awaitDriver = true
	}
</script>


<svelte:head>
	<title>That way: Passenger</title>
</svelte:head>

{#if !$session.user}
	<AcountAlert></AcountAlert>
{/if}

<main>
	<div class="float-btn" on:click={handleSignOut}>SignOut</div>

	<aside>
		
		{#if !awaitStartRoute}
			<form>
				<div class="form-group">
					<div class="inputs-container">
						<label for="">Home address</label>
						{#if !homeInCord}
							<input 
								bind:value={homeAddress} 
								type="text" 
								required={!homeInCord} 
								placeholder="Home address">
						{:else}
							<div class="cords-container">
								<input 
									type="number" 
									step="any"
									bind:value={homeAddress.lat}
									required={homeInCord} 
									placeholder="Latitude">
								<input 
									type="number" 
									step="any" 
									bind:value={homeAddress.long}
									required={homeInCord} 
									placeholder="Longitude">
							</div>
						{/if}
					</div>
				</div>
				
				<div class="form-group">
					<div class="inputs-container">
						<label for="">Destination address</label>
						{#if !destinationInCord}
							<input 
								bind:value={destinationAddress} 
								type="text" 
								required={!destinationInCord}
								placeholder="Destination address">
						{:else}
							<div class="cords-container">
								<input 
									type="number" 
									step="any"
									required={destinationInCord}
									placeholder="Latitude">
								<input 
									type="number" 
									step="any" 
									required={destinationInCord} 
									placeholder="Longitude">
							</div>
						{/if}
					</div>
				</div>

				<button on:click={handleSubmitForm} type='submit'>Go!</button>
			</form>	
		{:else if awaitStartRoute && !awaitDriver}
			<div class="await-to-start-route">
				<div>
					<p class="bold">Estimated time</p>
					<p>{estimatedTime}</p>
				</div>

				<div>
					<p class="bold">Estimated cost</p>
					<p>$ {estimatedCost}</p>
				</div>
				
				<div class="container-buttons">
					<button class="btn-important" on:click={handleRequest}>Request route</button>
					<button on:click={handleBack}>Back</button>
				</div>
			</div>
		{:else if awaitDriver && !routeStarted}
			<div >
				<h1>Waiting driver</h1>	
			</div>
		{:else if awaitDriver}
			<div>
				<h1>In route...</h1>
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

	.bold {
		font-weight: bold;
	}

	aside {
		position: absolute;
		z-index: 2;
		bottom: 0;
		left: 0;
		right: 0;
		background: #ffffff;
		box-shadow: 0 0 8px 0 grey;
	}

	form { 
		display: flex;
		flex-direction: column;
		padding: 0.5rem;
	}

	form button {
		padding: 1rem;
		font-size: 1.1rem;
		border-radius: 0.5rem;
		background: var(--blue-normal);
		border: none;
		outline: none;
		color: white;
		cursor: pointer;
	}

	.form-group {
		display: flex;
		justify-content: center;
		align-items: center;
		gap: 1rem;
	}
	
	.form-group label{
		display: block;
	}
	
	.form-group input[type=text],
	.form-group input[type=number] {
		width: 100%;
	}

	.form-group input[type=number] {
		flex-grow: 1;
	}

	.inputs-container {
		width: 100%;
		margin: 0.6rem 0;
	}

	.inputs-container input {
		padding: 0.3rem;
		margin: 0.3rem 0;
		border: none;
		outline: none;
		font-size: 1rem;
		border: 2px solid var(--blue-dark);
		border-radius: 0.3rem
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

	.await-to-start-route {
		padding: 0.5rem;
	}

	.await-to-start-route div {
		text-align: center;
		margin-top: 0.5rem;
	}

	.await-to-start-route .container-buttons {
		display: flex;
		flex-direction: column;
	}

	.await-to-start-route .container-buttons button {
		border: none;
		outline: none;
		cursor: pointer;
		margin: 0.5rem 0 0 0;
		background: gray;
		border-radius: 0.5rem;
		color: #000000;
		padding: 0.5rem;
		font-size: 0.8rem;
	}
	
	.await-to-start-route .container-buttons button.btn-important {
		color: white;
		background: var(--blue-normal);
		padding: 1rem;
		font-size: 1.1rem;
	}
</style>