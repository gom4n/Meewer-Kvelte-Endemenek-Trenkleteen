<!-- A post. Profile pictures not appearing while not logged in is intentional. -->

<script>
	import Container from "../lib/Container.svelte";
	import PFP from "../lib/PFP.svelte";
	import FormattedDate from "./FormattedDate.svelte";

	import {profileData, profileClicked, user, chatid, ulist, mainPage as page} from "../lib/stores.js";
	import {shiftHeld} from "../lib/keyDetect.js";
	import * as clm from "../lib/clmanager.js";
	
	import {onMount} from "svelte";
	export let post = {};

	// TODO: make bridged tag a setting

	/**
	 * Initialize this post's user profile - gets profile info from the cache or fetches it.
	 */
	function initPostUser() {
		if (!post.user) return;
		if (!($user.name)) return;

		var userName = ""
		var userName2 = ""
		if (post.user == "Discord" && post.content.includes(":")) {
			post.user = post.content.split(": ")[0];
			post.content = post.content.slice(post.content.indexOf(": ")+1);
		}
		post.content = post.content.replaceAll("i", "e")
		post.content = post.content.replaceAll("a", "e")
		post.content = post.content.replaceAll("o", "e")
		post.content = post.content.replaceAll("u", "e")
		post.content = post.content.replaceAll("I", "E")
		post.content = post.content.replaceAll("A", "E")
		post.content = post.content.replaceAll("U", "E")
		post.content = post.content.replaceAll("O", "E")
		post.content = post.content.replaceAll("s", "k")
		post.content = post.content.replaceAll("S", "K")
		userName = post.user;
		userName2 = userName
		userName2 = userName2.replaceAll("i", "e")
		userName2 = userName2.replaceAll("a", "e")
		userName2 = userName2.replaceAll("o", "e")
		userName2 = userName2.replaceAll("u", "e")
		userName2 = userName2.replaceAll("I", "E")
		userName2 = userName2.replaceAll("A", "E")
		userName2 = userName2.replaceAll("U", "E")
		userName2 = userName2.replaceAll("O", "E")
		userName2 = userName2.replaceAll("S", "k")
		userName2 = userName2.replaceAll("S", "K")
		post.userName2 = userName2

		/**
		 * Fetch the user profile and store it in the cache.
		 */
		const getProfile = () => {
			let _profileData = $profileData;

			if (userName === "Notification") {
				_profileData[userName] = {
					pfp_data: 101
				}
				profileData.set(_profileData);
				return;
			} else if (userName === "Announcement") {
				_profileData[userName] = {
					pfp_data: 102
				}
				profileData.set(_profileData);
				return;
			} else if (userName === "Server") {
				_profileData[userName] = {
					pfp_data: 102
				}
				profileData.set(_profileData);
				return;
			}

			_profileData[userName] = {
				pfp_data: -1,
			};
			profileData.set(_profileData);

			clm.meowerRequest({
				cmd: "direct",
				val: {
					cmd: "get_profile",
					val: userName,
				},
				listener: "get_profile_" + userName,
			}).then(val => {
				// Ding dong! The data has arrived.
				_profileData[userName] = val.payload;
				profileData.set(_profileData);
			}).catch(e => {
				// Uh oh - something has gone wrong.
				_profileData[userName] = {
					error: true,
					pfp_data: -2,
					temporary: true,
				};
				profileData.set(_profileData);
			})
		}

		// Do we have a stored profile?
		const _profileData = $profileData;
		if (_profileData[userName]) {
			// Reuse the cached data if the profile isn't temporary
			if (_profileData[userName].temporary) {
				getProfile();
			}
		} else {
			// Get the profile!
			getProfile();
		}
	};
	onMount(initPostUser);
</script>

<Container>
	<div class="post-header">
		<div class="settings-controls">
			{#if $user.name && $chatid !== "livechat" && post.user !== "Server"}	
			{#if $user.lvl >= 1 || post.user === $user.name}
			<button
				class="circle close"
				on:click={()=>{
					if (shiftHeld || confirm("Are you sure you want to delete this post?")) {
						clm.meowerRequest({
							cmd: "direct",
							val: {
								cmd: "delete_post",
								val: post.post_id,
							},
						});
					}
				}}
			></button>
			{:else}
			<button
				class="circle report"
				on:click={()=>{
					if (confirm("Are you sure you want to report this post?")) {
						clm.meowerRequest({
							cmd: "direct",
							val: {
								cmd: "report",
								val: {
									type: 0,
									id: post.post_id,
								},
							},
						});
					}
				}}
			></button>
			{/if}
			{/if}
		</div>
		<button 
			class="pfp" 
			on:click={()=>{
				if (post.user === "Notification" || post.user === "Announcement") return;
				profileClicked.set(post.user);
				page.set("profile");
			}}
		>
			<PFP
				icon={$profileData[post.user] ? $profileData[post.user].pfp_data : -3}
				alt="{post.user}'s profile picture"
				online={$ulist.includes(post.user)}
			></PFP>
		</button>	
		<div class="creator">
			<h2 class="creator">{post.userName2}</h2>
			<FormattedDate date={post.date}></FormattedDate>
		</div>
	</div>
	<p class="post-content">{post.content}</p>
</Container>

<style>
	.pfp {
		margin-right: 0.2em;
		padding: 0;
		border: none;
		background: none !important;
		color: inherit;
	}
	.post-header {
		display: flex;
		align-items: center;
		flex-wrap: wrap;
	}
	.creator {
		display: inline;
		max-width: 100%;
		margin-left: 0.5em;
	}
	.creator h2 {
		display: block;
		font-size: 200%;
		margin: 0;
	}
	.pfp:hover:not(:active) :global(.pfp), .pfp:focus-visible :global(.pfp) {
		transform: scale(1.1);
	}
	.post-content {
		white-space: pre-wrap;
	}
	.settings-controls {
		position: absolute;
		top: 0.25em;
		right: 0.25em;
	}
	button.circle {
		border: none;
		margin: 0;
		margin-left: 0.125em;
	}
</style>
