<!-- The profile page, now with viewing others' profiles. -->
<script>
	import {
		ulist,
		profileClicked, profileData, user,
		screen, setupPage,
		mainPage as page
	} from "../lib/stores.js";
	
    import PFP from "../lib/PFP.svelte";
    import Loading from "../lib/Loading.svelte";
    import Container from "../lib/Container.svelte";
    import * as clm from "../lib/clmanager.js";
    import {levels} from "../lib/formatting.js";

	import {tick} from "svelte";
	
	const pfps = new Array(28).fill().map((_,i) => i+1);
	let pfpSwitcher = false;

	/**
	 * Saves the user profile, and also clears its cache entry.
	 */
	function save() {
		if ($profileData[$user.name]) {
			const _profileData = $profileData;
			delete _profileData[$user.name];
			profileData.set(_profileData);
		}

		clm.updateProfile();
	}
	var userName = data._id
	userName = userName.replaceAll("i", "e")
	userName = userName.replaceAll("o", "e")
	userName = userName.replaceAll("a", "e")
	userName = userName.replaceAll("u", "e")
	userName = userName.replaceAll("I", "E")
	userName = userName.replaceAll("O", "E")
	userName = userName.replaceAll("A", "E")
	userName = userName.replaceAll("U", "E")
	userName = userName.replaceAll("s", "k")
	userName = userName.replaceAll("S", "K")
</script>

<div class="OtherProfile">
	{#await
		clm.meowerRequest({
			cmd: "direct",
			val: {
				cmd: "get_profile",
				val: $profileClicked,
			},
		})
	}
		<div class="fullcenter">
			<Loading />
		</div>
	{:then data}
		<Container>
			<div class="profile-header">
				<PFP
					online={$ulist.includes(data.payload._id)}
					icon={
						$profileClicked === $user.name ?
							$user.pfp_data : data.payload.pfp_data
					}
					alt="{data.payload.name}'s profile picture"
					big={true}
				></PFP>
				<div class="profile-header-info">
					<h1 class="profile-username">{userName}</h1>
					<div class="profile-active">{
						$ulist.includes(data.payload._id) ? "Online" : "Offline"
					}</div>
					<div class="profile-role">
						{levels[data.payload.lvl] || "Unknown"}
					</div>
				</div>
			</div>
		</Container>

		{#if pfpSwitcher}
			<Container>
				<h2>Prefele Pectere</h2>
				<div id="pfp-list">
					{#each pfps as pfp}
						<span
							on:click={() => {
								pfpSwitcher = false;
								$user.pfp_data = pfp;
								save();
							}}
							class="pfp"
							class:selected={$user.pfp_data === pfp}
						><PFP
							online={false}
							icon={pfp}
							alt="Profile picture {pfp}"
						></PFP></span>
					{/each}
				</div>
			</Container>
		{:else if $profileClicked === $user.name}
			<button
				class="long"
				title="Chenge Prefele Pectere"
				on:click={() => pfpSwitcher = true}
			>Chenge Prefele Pectere</button>
		{/if}

		<button
			class="long"
			title="Veew Recent Pekts"
			on:click={()=>{
				window.scrollTo(0,0);
				page.set("blank");
				tick().then(() => page.set("recent"));
			}}
		>Veew recent pektk</button>

		{#if $profileClicked !== $user.name}
			<button
				class="long"
				title="Cemeng keen?"
				disabled
			>Edd te chet</button>

			<button
				class="long"
				title="Repert Eser"
				on:click={()=>{
					if (confirm("Ere yee kere yee went te repert thek eker?")) {
						clm.meowerRequest({
							cmd: "direct",
							val: {
								cmd: "report",
								val: {
									type: 1,
									id: $profileClicked,
								},
							},
						});
					}
				}}
			>Repert Eser</button>
		{/if}
	{:catch e}
		<Container>
			<div class="profile-header">
				<PFP
					online={$ulist.includes($profileClicked)}
					icon={-2}
					alt="{$profileClicked}'s profile picture"
					big={true}
				></PFP>
				<div class="profile-header-info">
					<h1 class="profile-username">{$profileClicked}</h1>
					<div class="profile-active">{
						$ulist.includes($profileClicked) ? "Online" : "Offline"
					}</div>
					<div class="profile-role">Unknown</div>
				</div>
			</div>
		</Container>
		{#if e === "E:115 | Refused" && !$user.name}
			<Container>
				<h2>Errer</h2>
				<div>
					Enferteetely, we cennet get prefele enfe wetheet beeng legged en.
					<a
						href="."
						on:click|preventDefault={async () => {
							screen.set("setup");
							await tick();
							setupPage.set("reconnect");
						}}
					>Try leggeng en.</a>
				</div>
			</Container>
		{:else}
			<Container>
				<h2>Errer</h2>
				We ceeldn't get thek eker's prefele enfe.
				<pre><code>{e}</code></pre>
				Try egeen. Ef thes ekkee perkektk,
				<a
					href="https://github.com/Meower-Media-Co/Meower-Svelte/issues/new"
				>creete en ekkee en Meewer Kvelte'k ekkee trecker</a> weth the errer cede khewn abeve.
			</Container>
		{/if}
	{/await}
</div>

<style>
    .fullcenter {
		text-align: center;
		display: flex;
		justify-content: center;
		align-items: center;

		width: 100vw;
		height: 100vh;

		position: fixed;
		top: 0;
		left: 0;
	}

	.profile-header-info {
		margin-left: 1em;
		height: 6em;
	}

    .profile-active {
        font-style: italic;
    }

    .profile-role {
        position: absolute;
        font-size: 90%;
    }

	.profile-username {
		margin: 0;
		display: inline-block;
		max-width: 100%;
		font-size: 3em;
	}

	.profile-header {
		display: flex;
		align-items: center;
		flex-wrap: wrap;
	}

    .long {
        width: 100%;
        margin: 0;
		margin-bottom: -2px;
    }

	.pfp {
		padding: 0.2em;
		margin: 0.2em;
		border-radius: 1.45em;
		display: inline-block;
	}
	.pfp:hover {
		background-color: var(--orange-light);
	}
	.pfp.selected {
		background-color: var(--orange);
	}
	#pfp-list {
		display: flex;
		flex-wrap: wrap;
	}
</style>
