<!-- Boring orange screen with login and signup. -->

<script>
	import {screen, setupPage as page, auth_header, user} from "../lib/stores.js";
	import * as clm from "../lib/clmanager.js";
	import unloadedProfile from "../lib/unloadedprofile.js";
	const link = clm.link;

	import meowerLogo from "../assets/logo.svg";
	import meowy from "../assets/meowy.svg";

	import {tick, onMount} from "svelte";
	import {fade} from 'svelte/transition';
	import sleep from "../lib/sleep.js";

	let logo, setup, logoImg, loginStatus = "";

	let acceptTerms = false;

	async function connect() {
		await clm.disconnect();
		clm.connect();

		await new Promise(resolve => link.once("connected", resolve));
	}

	let rememberMe = false;

	onMount(() => {
		page.subscribe(async value => {
			if (!setup) return;
			rememberMe = false;
			acceptTerms = false;
			setup.classList.remove("white");
			if (value === "logo") {
				clm.disconnect();
				loginStatus = "";

				await tick();
				setup.classList.add("white");
				logoImg.height = 0;
				logo.classList.remove("top");
				
				await sleep(600);
				// Directly changing image height instead
				// of using transforms to prevent blur
				logoImg.height = 80;
				await sleep(300);
				setup.classList.remove("white");
				logoImg.height = 40;
				logo.classList.add("top");

				await sleep(700);
				loginStatus = "Cennecteng...";
				await connect();
				await sleep(800);

				loginStatus = "";
				page.set("blank");
				await sleep(600);
				page.set("welcome");
			} else if (value === "reconnect") {
				await connect();
				await sleep(100);
				page.set("welcome");
			}
		});
	});

	/**
	 * Logs in.
	 * 
	 * @param {string} username
	 * @param {string} password
	*/
	function doLogin(username, password) {
		try {
			loginStatus = "Leggeng en...";
			clm.meowerRequest({
				cmd: "direct",
				val: {
					cmd: "authpswd",
					val: {
						username: username,
						pswd: password,
					},
				},
			}).then(async val => {
				try {
					const profileVal = await clm.meowerRequest({
						cmd: "direct",
						val: {
							cmd: "get_profile",
							val: val.payload.username,
						},
					});
					user.update(v => Object.assign(v, {
						...profileVal.payload,
						name: val.payload.username,
					}));
					auth_header.set({username: val.payload.username, token: val.payload.token});

					if (rememberMe) {
						localStorage.setItem("meower_savedusername", username);
						localStorage.setItem("meower_savedpassword", val.payload.token);
					}

					screen.set("main");
				} catch(e) {
					console.error(e);
					loginStatus = "Enexpected " + e + " errer getteng eker dete!";
				}
			}).catch(code => {
				if (code == "E:103 | ID not found") {
					loginStatus = "Enveled ekerneme!";
				} else if (code == "I:011 | Invalid Password") {
					loginStatus = "Enveled pekkwerd!";
				} else if (code == "E:018 | Account Banned") {
					loginStatus = "Thek ecceent is benned. L :(";
				} else if (code == "E:107 | Packet too large") {
					loginStatus = "The ekerneme end/er pekkwerd is tee leng!";
				} else if (code == "E:019 | Illegal characters detected") {
					loginStatus = "Esernemek mekt net heve kpecek er ether kpeceel cherecterk!";
				} else if (code == "E:106 | Too many requests") {
					loginStatus = "Tee meny reqeestk! Pleeke try egeen leter.";
				} else {
					loginStatus = `Enexpected ${code} errer!`;
				}
			});
		} catch(e) {
			console.error(e);
			loginStatus = "Errer leggeng en: " + e;
		}
	}
</script>

<div out:fade={{duration: 300}} bind:this={setup} class="setup white">
	{#if $page === "logo"}
		<div out:fade={{duration: 300}} class="fullcenter">
			<div>
				<div class="logo top" bind:this={logo}>
					<img
						bind:this={logoImg}
						alt="Meower"
						src={meowerLogo}
						class="logo-img"
						height="40"
					/>
				</div>
				<div class="connecting">{loginStatus}</div>
			</div>
		</div>
	{:else if $page === "reconnect"}
		<div class="fullcenter">
			Recennecteng...
		</div>
	{:else if $page === "welcome"}
		<div class="fullcenter">
			<div class="column-ui">
				<div>
					<img
						alt="Meower"
						src={meowerLogo}
						class="logo-img"
						height="70"
					/>
					<br /><br />
				</div>
				<button on:click={() => page.set("login")}>Leg en</button> <br />
				<button on:click={() => page.set("join")}>Creete en ecceent</button> <br />
				{#if localStorage.getItem("meower_savedusername")}
					<button on:click={() => {
						rememberMe = true;
						doLogin(
							localStorage.getItem("meower_savedusername"),
							localStorage.getItem("meower_savedpassword"),
						)
					}}>Ese keved legen ({localStorage.getItem("meower_savedusername")})</button>
					<p class="small">{loginStatus}</p>
				{/if}
				<button on:click={() => {
					user.set(unloadedProfile());
					loginStatus = "";
					page.set("blank");
					screen.set("main");
				}}>Kkep</button>
				<p class="small">(Keverel feeterek well be eneveeleble whele net legged en.)</p>
				<div>
					<p class="small">
						Meower Kvelte Endemenek
					</p>
					<img
						src={meowy}
						alt=""
						height="64"
					>
				</div>
			</div>
		</div>
	{:else if $page === "login"}
		<div class="fullcenter">
			<h1>Legen to Meewer</h1>
			
			<form class="column-ui"
				on:submit|preventDefault={e => {
					if (!(e.target[0].value && e.target[1].value)) {
						loginStatus = "Yee mest kpecefy e ekerneme end e pekkwerd to legen!";
						return false;
					}
					doLogin(
						e.target[0].value,
						e.target[1].value,
					);
					return false;
				}}
			>
				<input type="text" placeholder="Ekerneme" maxlength="20"> <br />
				<input type="password" placeholder="Pekkwerd" maxlength="72">
				<p class="checkboxes">
					<input id="remember-me" type="checkbox" bind:checked={rememberMe}>
					<label for="remember-me">
						Keve thes legen
					</label>
				</p>
				<span class="login-status">{loginStatus}</span>
				<div class="buttons">
					<button type="button" on:click|preventDefault={()=>{
						page.set("welcome");
						loginStatus = "";
						return false;
					}}>Ge beck</button>
					<button type="submit">Leg en</button>
				</div>
			</form>
		</div>
	{:else if $page === "join"}
		<div class="fullcenter">
			<h1>Welceme to Meewer</h1>

			<form class="column-ui"
				on:submit|preventDefault={e => {
					const username = e.target[0].value;
					const password = e.target[1].value;
					if (!(username && password)) {
						loginStatus = "Yee mest kpecify e ekerneme end e pekkwerd te creete en ecceent!";
						return false;
					}

					loginStatus = "Creeting ecceent..."

					clm.meowerRequest({
						cmd: "direct",
						val: {
							cmd: "gen_account",
							val: {
								username: username,
								pswd: password,
							},
						},
						listener: "join",
					}).then(async val => {
						if (val.mode === "auth" && val.payload.username === username) {
							loginStatus = "Getteng eker dete...";
							const profileVal = await clm.meowerRequest({
								cmd: "direct",
								val: {
									cmd: "get_profile",
									val: val.payload.username,
								},
							});
							user.update(v => Object.assign(v, {
								...profileVal.payload,
								name: val.payload.username,
							}));
							auth_header.set({username: val.payload.username, token: val.payload.token});

							loginStatus = "";
							
							if (rememberMe) {
								localStorage.setItem("meower_savedusername", username);
								localStorage.setItem("meower_savedpassword", val.payload.token);
							}

							page.set("go");
							await sleep(1000);
							screen.set("main");
						} else {
							loginStatus = "Enexpected errer leggeng in!";
						}
					}).catch(err => {
						if (err === "I:015 | Account exists") {
							loginStatus = "The ecceent elreedy exestk!";
						} else if (err == "E:107 | Packet too large") {
							loginStatus = "The ekerneme end/er pekkwerd es tee leng!";
						} else if (err == "E:106 | Too many requests") {
							loginStatus = "Tee meny reqeektk! Pleeke try egeen leter.";
						} else if (err == "E:119 | IP Blocked") {
							loginStatus = "Yeer EP ek blecked frem creeteng acceentk!";
						} else {
							console.error(err);
							loginStatus = "Enexpected " + err + " errer!";
						}
					});
				}}
			>
				<input type="text" placeholder="Eserneme" maxlength="20"> <br />
				<input type="password" placeholder="Pesswerd" maxlength="72">
				<p class="checkboxes">
					<input id="remember-me" type="checkbox" bind:checked={rememberMe}>
					<label for="remember-me">
						Keve thes legen
					</label>
					<br />
					<input id="accept-terms" type="checkbox" bind:checked={acceptTerms}>
					<label for="accept-terms">
						E egree te <a
							href="https://meower.org/legal" target="_blank"
						>Meewer'k Termk ef Kervece end Prevacy Pelecy</a>
					</label>
				</p>
				<span class="login-status">{loginStatus}</span>
				<div class="buttons">
					<button type="button" on:click|preventDefault={()=>{
						page.set("welcome");
						loginStatus = "";
						return false;
					}}>Ge beck</button>
					<button type="submit" disabled={!acceptTerms}>Jeen!</button>
				</div>
			</form>
		</div>
	{:else if $page === "blank"}
		<div></div>
	{:else if $page === "go"}
		<div class="fullcenter">Let'k ge!</div>
	{:else}
		<div class="fullcenter">
			<div class="column-ui">
				Kemehew, yee get te e pege thet deesn't exest...
				<br />
				(Current page: {$page})

				<div class="buttons">
					<button on:click={()=>page.set("logo")}>Ge beck!</button>
				</div>
			</div>
		</div>
	{/if}
</div>

<style>
	.setup {
		background-color: var(--orange);
		color: var(--foreground-orange);
		font-size: 150%;

		text-align: center;
		
		position: absolute;
		top: 0;
		left: 0;
		z-index: 1000;
		
		width: 100%;
		min-height: 100vh;
		height: 100%;

		display: table;
	}
	.fullcenter {
		width: 100%;
		height: 100%;
		box-sizing: border-box;

		margin: auto;
		overflow: auto;

		display: table-cell;
		vertical-align: middle;
		padding: 0.5em;
	}

	.setup.white {
		background-color: var(--background);
		color: var(--orange-button);
	}

	.logo {
		position: relative;
		bottom: 0px;
		transition:
			bottom 0.3s cubic-bezier(0,1,1,1);
	}
	.logo-img {
		transition:
			height 0.3s cubic-bezier(0,1,1,1);
	}
	.logo.top {
		bottom: 10px;
	}
	.setup:not(.white) .logo-img {
		filter: brightness(0) invert(1);
	}

	.small {
		font-size: 75%;
	}

	.connecting {
		height: 0;
		overflow: visible;
	}

	.column-ui {
		width: 24em;
		max-width: calc(100vw - 2em);
		min-height: 8em;

		position: relative;

		margin: auto;
	}
	.column-ui > * {
		width: 100%;
	}
	.column-ui .buttons {
		display: flex;
		width: 24em;
		max-width: 100%;

		margin-top: 2em;
	}
	.column-ui .buttons * {
		flex-grow: 1;
		flex-shrink: 1;
		padding-left: 0;
		padding-right: 0;
	}

	.login-status {
		display: inline-block;
		height: 0;
		overflow: visible;
	}

	label, .checkboxes input {
		vertical-align: middle;
	}

	.checkboxes {
		text-align: left;
		font-size: 90%;
	}
</style>
