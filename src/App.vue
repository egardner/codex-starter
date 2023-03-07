<template>
	<header class="app-header">
		<nav>
			<h1>Nearby Wiki Articles</h1>
		</nav>
	</header>	

	<main class="app-main">
		<!-- 
			This is the results template. It will be shown to the user
			when the API request is complete.
		-->
		<div v-if="articles" class="app-results">
			<!-- If any nearby articles have been found, they will be shown here -->
			<ul v-if="articles.length > 0">
				<li v-for="article in articles">
					<!-- Right now we just show the title for each  -->
					<p>
						{{ article.title }}
					</p>
				</li>
			</ul>

			<!-- If no articles are found, this message will be shown to the user -->
			<p v-else>
				Sorry, no nearby articles found.
			</p>

			<!-- This button allows the user to reset the page -->
			<p>
				<cdx-button 
					class="btn-clear"
					action="destructive"
					@click="resetArticles" 
				>
					Clear current results
				</cdx-button>
			</p>
		</div>

		<!-- 
			This is the starting template. It shows buttons that allow the user
			to choose what they want to see. 
		-->
		<div v-else>
			<p>
				<cdx-button 
					class="btn-nearby"
					action="progressive" 
					type="primary"
					@click="showNearbyArticles" 
				>
					<cdx-icon :icon="cdxIconMapPin" size="small" />
					Show nearby articles
				</cdx-button>
			</p>

			<p>
				<cdx-button 
					class="btn-random"
					action="progressive"
					@click="showRandomArticles" 
				>
					<cdx-icon :icon="cdxIconDie" size="small" />
					Show random articles
				</cdx-button>
			</p>

		</div>
	</main>
</template>

<script>
import { CdxButton, CdxIcon } from '@wikimedia/codex';
import { cdxIconMapPin, cdxIconDie } from '@wikimedia/codex-icons';

export default {
	components: {
		CdxButton,
		CdxIcon
	},

	data() {
		return {
			articles: null, // this becomes an Array when the API request is complete
			cdxIconDie,
			cdxIconMapPin
		};
	},

	methods: {
		/**
		 * Get a list of articles based on the user's geolocation (if they allow it)
		 */
		showNearbyArticles() {
			navigator.geolocation.getCurrentPosition( 
				success => {
					this.fetchArticlesByGeolocation( success.coords.latitude, success.coords.longitude );
				},
				error => {
					console.log( error );
				}
			)
		},

		/**
		 * Get a list of articles based on a random geolocation
		 */
		showRandomArticles() {
			this.fetchArticlesByGeolocation(
				'0.000000000000000',
				'0.000000000000000'
			);
		},

		/**
		 * Get a list of articles based on coordinates. This method talks to
		 * Wikipedia's API using a built-in function called "fetch"; fetch is
		 * included in most modern web browsers, and can be used to retreive
		 * data from the web.
		 *
		 * @param {string} lat 
		 * @param {string} lng 
		 */
		fetchArticlesByGeolocation( lat, lng ) {
			const base = 'https://en.wikipedia.org/w/api.php';
			const params = {
				action: 'query',
				format: 'json',
				origin: '*',
				formatversion: 2,
				prop: 'coordinates|pageprops|pageimages|description|info|pageterms',
				inprop: 'url',
				colimit: 'max',
				generator: 'geosearch',
				ggsradius: 10000,
				ggsnamespace: 0,
				ggslimit: 25,
				redirects: 'no',
				uselang: 'en',
				ppprop: 'displaytitle',
				piprop: 'thumbnail',
				pithumbsize: 150,
				pilimit: 50,
				codistancefrompoint: `${lat}|${lng}`,
				ggscoord: `${lat}|${lng}`
			};
			
			const apiUrl = new URL( base );
			apiUrl.search = new URLSearchParams( params );

			// Open your network tab in your browser's developer tools
			// to see this request being made.
			fetch( apiUrl )
				.then( response => response.json() ) // parse the response JSON into a data object
				.then( data => data.query.pages ) // get the pages array out of the data object
				.then( pages => {
					this.articles = pages; // stash the pages data into our component's data as "articles" 

					// Uncomment the line below to see the data that was fetched in your browser's console.
					// console.log( pages );
				} );
		},

		/**
		 * Clear the article data
		 */
		resetArticles() {
			this.articles = null;
		}
	}
}
</script>

<style>
html, body {
	font-family: var( --font-family-base );
}

.app-header {
	background-color: var( --background-color-base );
	display: flex;
	align-items: center;
	justify-content: center;
	padding: var( --spacing-25 ) var( --spacing-75 );
	margin-top: var( --spacing-100 );
	position: sticky;
	top: 0px;
}

.app-header h1 {
	margin: 0;
}

.app-main {
	display: flex;
	flex-direction: column;
	align-items: stretch;
	justify-content: center;
	padding: var( --spacing-25 ) var( --spacing-75 );
	text-align: center;
}

.app-results ul {
	text-align: left;
}

.btn-nearby,
.btn-random,
.btn-clear {
	width: var( --size-full );
}

</style>