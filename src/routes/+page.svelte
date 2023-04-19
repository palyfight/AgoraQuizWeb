<script>
	const MONTH_NAMES_FR = [
	  "Janvier",
	  "Février",
	  "Mars",
	  "Avril",
	  "Mai",
	  "Juin",
	  "Juillet",
	  "Août",
	  "Septembre",
	  "Octobre",
	  "Novembre",
	  "Décembre"
	];
	const BASE_URL = "http://localhost:1337"
	const API_URL = `${BASE_URL}/api`;

	let themePromise;
	let filteredThemePromise;
	let upcomingThemePromise;
	let selectedMonth;
	let rankingTable;

	function convertDateToMonthNameAndYear(dateToConvert) {
		const [year, month, day] = dateToConvert.split("-");
		const monthName = MONTH_NAMES_FR[parseInt(month) - 1];

		return `${monthName} ${year}`
	}

	function isDateWithinSelectedMonth(dateStr) {
	  	const [year, month, day] = dateStr.split("-");
	  	const [selectYear, selectMonth] = selectedMonth.split("-");

	  	return month === selectMonth;
	}

	// Thank you ChatGPT :)
	function getCurrentDate() {
		const currentDate = new Date();
		const year = currentDate.getFullYear();
		const month = String(currentDate.getMonth() + 1).padStart(2, '0');
		const day = String(currentDate.getDate()).padStart(2, '0');
		const formattedDate = `${year}-${month}-${day}`;

		return formattedDate;
	}

	const filterThemesByMonth = async (month) => {
		var foo = await themePromise;
		filteredThemePromise = foo.data.filter(obj => isDateWithinSelectedMonth(obj.attributes.triviaDate));
	}
	
	const fetchAvailableThemeList = async () => {
		var now = getCurrentDate();
		var response = await fetch(`${API_URL}/themes?populate=*&filters[$and][0][triviaDate][$lte]=${now}`);
		var result = await response.json();
		return result;
	}

	const getUpcomingThemes = async () => {
		var now = getCurrentDate();
		var response = await fetch(`${API_URL}/themes?populate=*&pagination[limit]=3&filters[triviaDate][$gt]=${now}`);
		var result = await response.json();
		return result;
	}

	themePromise = fetchAvailableThemeList();
	upcomingThemePromise = getUpcomingThemes();
</script>

<!-- Wrapper -->
<div id="wrapper">

	<!-- Header -->
		<header id="header" class="alt">
			<span class="logo"><img src="images/branding/logo_white_transparent.png" alt="" /></span>
			<p>Tous les Lundi à 18:30 au pub <a href="https://www.agora.pelemele.ca/">Pêle-Mêle - Agora</a></p>
		</header>

	<!-- Nav -->
		<nav id="nav">
			<ul>
				<li><a href="#intro" class="active">Classement des équipes</a></li>
				<li><a href="#first">Thèmes à venir</a></li>
			</ul>
		</nav>

	<!-- Main -->
		<div id="main">

			<!-- Introduction -->
				<section id="intro" class="main">
					<header class="major">
						<p>
							{#if themePromise}
								<select bind:value={selectedMonth} on:change={(event) => filterThemesByMonth(selectedMonth)}>
									<option value="">- Mois -</option>
									{#await themePromise}
										<option>Loading....</option>
									{:then themes}
										{#each themes.data as theme, i}
											<option value="{theme.attributes.triviaDate}">{convertDateToMonthNameAndYear(theme.attributes.triviaDate)}</option>
										{/each}
									{:catch err}
										<option>Oops!</option>
									{/await}
								</select>
							{/if}
						</p>
						<p>
							{#if filteredThemePromise}
								<select bind:value={rankingTable}>
									<option value="">- Thème -</option>
									{#await filteredThemePromise}
										<option>Loading....</option>
									{:then themes}
										{#each themes as theme, i}
											<option value="{theme.attributes.ranking.data.attributes.table}">{theme.attributes.name}</option>
										{/each}
									{:catch err}
										<option>Oops!</option>
									{/await}
								</select>
							{/if}
						</p>
					</header>

					<div class="spotlight">
						{#if rankingTable}
							<div class="content table-wrapper">
								{@html rankingTable}
							</div>

							<span class="image"><img src="images/trophy.png" alt="" /></span>
						{/if}
					</div>
				</section>

			<!-- First Section -->
				<section id="first" class="main special">
					<header class="major">
						<h2>Prochains thèmes</h2>
					</header>
					<ul class="features">
						{#await upcomingThemePromise}
							<li>Chargement....</li>
						{:then trivias}
							{#each trivias.data as trivia}
								<li>
									<span class="image">
										<img src="{BASE_URL + trivia.attributes.photo.data.attributes.formats.thumbnail.url}" alt=""/>
									</span>
									<h3>{trivia.attributes.name}</h3>
									<p>{trivia.attributes.triviaDate}</p>
								</li>
							{/each}
						{:catch err}
							<option>Oops!</option>
						{/await}
					</ul>
				</section>
		</div>

	<!-- Footer -->
		<footer id="footer">
			<section>
				<dl class="alt">
					<dt>Addresse</dt>
					<dd>
						20 Allée de Hambourg #100 &bull; Gatineau &bull; QC &bull; J9J 4J6
					</dd>
				</dl>
			</section>

			<p class="copyright">&copy; Fait avec <i class="icon fa-solid fa-heart" style="color: #ffffff;"></i> par Christian Bamatembera.</p>
		</footer>

</div>