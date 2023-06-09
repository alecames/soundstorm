<script>
	// @ts-nocheck

	import { formatNumber, relativeTime, readableNumber, absoluteTime } from '$lib/utils';
	import { playing } from '$lib/state';
	import supabase from '$lib/supabase';
	import { onMount } from 'svelte';
	import { slide } from 'svelte/transition';
	import { user } from '$lib/auth';
	import { writable } from 'svelte/store';

	export let track = {
		id: 0,
		title: '',
		author: '',
		likes: 0,
		plays: 0,
		duration: 0
	};

	/**
	 * @type {string | any[]}
	 */
	let comments = [];
	let comment = '';
	let liked = 0;
	let overflowMenu = false;

	const submitComment = async (e) => {
		e.preventDefault();

		if (comment.length > 0) {
			const { data, error } = await supabase.from('comments').insert([
				{
					author: $user.username,
					content: comment,
					related_track: track.id
				}
			]);

			if (error) {
				console.error('Error submitting comment:', error);
			} else {
				comment = '';
			}
		}
		await fetchComments();
	};

	const likeTrack = async () => {};

	function repost() {
		console.log('repost');
	}

	function share() {
		console.log('share');
	}

	function showOptions() {
		overflowMenu = !overflowMenu;
	}

	async function fetchComments() {
		const { data, error } = await supabase
			.from('comments')
			.select('author, content, created_at')
			.eq('related_track', track.id);

		if (error) {
			console.error('Error fetching comments:', error);
		} else {
			comments = data;
		}
	}

	onMount(async () => {
		fetchComments();
	});
</script>

<div class="card">
	{#if track && track.id}
		<div class="content">
			<div class="top-container">
				<div class="play-button">
					{#if true}
						<span
							on:click={() => ($playing = true)}
							on:keydown
							class="material-symbols-outlined icon play-icon">play_circle</span
						>
					{:else}
						<span
							on:click={() => ($playing = false)}
							on:keydown
							class="material-symbols-outlined icon play-icon">pause_circle</span
						>
					{/if}
				</div>
				<div>
					<h2 class={`${playing ? 'playing' : ''}`}>
						<span class="accent">{track.author ? track.author : 'anonymous'}</span> - {track.title}
						<span class="date-posted" title={absoluteTime(track.created_at)}
							>{relativeTime(track.created_at)}</span
						>
					</h2>
				</div>
			</div>
			{#if track.description}
				<p class="desc">{track.description}</p>
			{/if}

			<form class="interactions" on:submit={submitComment}>
				<div class="comment-section">
					{#if comments && comments.length > 0}
						<hr class="track-divider" />
						<div class="comment-list">
							{#each comments as comment}
								<div class="comment" in:slide>
									<p class="comment-author">{comment.author}</p>
									<p class="comment-text">{comment.content}</p>
									<p class="comment-date" title={absoluteTime(comment.created_at)}>
										&nbsp;•&nbsp;{relativeTime(comment.created_at)}
									</p>
								</div>
							{/each}
						</div>
					{:else}
						<div class="comment-list">
							<p class="no-comments">No comments yet. Be the first to comment on this track!</p>
						</div>
					{/if}
					<input
						name="comment"
						bind:value={comment}
						disabled={!$user || !$user.id}
						class="comment-textfield"
						type="text"
						autoCorrect="off"
						placeholder={$user && $user.id
							? 'Write a comment...'
							: 'You must be logged in to comment'}
					/>
				</div>
				<div class="stats">
					<div class="views" title="{readableNumber(track.views)} views">
						<span class="material-symbols-rounded icon">play_arrow</span>
						<span>{formatNumber(track.views)}</span>
					</div>
					<div
						class="likes"
						on:click={likeTrack}
						on:keydown
						title="{readableNumber(track.likes)} likes"
					>
						<span class={`material-symbols-rounded icon ${liked ? 'active' : ''}`}> favorite </span>
						<span>{formatNumber(track.likes)}</span>
					</div>
					<span on:click={share} on:keydown class="material-symbols-rounded icon">share</span>
					<span on:click={showOptions} on:keydown class="material-symbols-rounded icon"
						>more_vert</span
					>
				</div>
			</form>
		</div>
	{/if}
</div>

<style lang="sass">
@use '/src/variables.sass' as *

.card
	display: flex
	flex-flow: column
	justify-content: left
	align-items: center
	flex: 0 0 auto
	width: 100%
	padding: 1rem
	background-color: $primary
	border: 1px solid $border
	border-radius: 12px
	transition: background-color 0.3s $curve

	.top-container
		display: flex
		flex-flow: row wrap
		justify-content: left
		align-items: center
		gap: 1rem

	.date-posted
		color: fade-out($text, 0.5)
		vertical-align: center
		font-size: 0.8rem
		margin-left: 0.5rem

	.content
		width: 100%

		.title
			font-size: 1.5rem
			margin-bottom: 0.5rem
			padding: 0

		.desc
			margin-bottom: 0.5rem
			color: $text

	.track-divider
		border: none
		margin: 1rem 0
		border-top: 1px solid $border

	.interactions
		display: flex
		gap: 1rem
		flex-flow: row wrap
		justify-content: space-between
		align-items: flex-end

		.comment-section
			display: flex
			flex-direction: column
			gap: 0.5rem
			flex: 1 1 50%

			p
				margin-bottom: 0
				padding-bottom: 0

		.stats
			display: flex
			gap: 1rem
			align-items: center
			margin-bottom: 0.25rem
			flex: 0.5

	.comment-list
		margin-bottom: 1rem
		display: flex
		flex-direction: column

	.no-comments
		color: fade-out($text,0.4)

	.comment
		padding: 3px 10px
		margin: 0
		background: none
		display: flex
		flex-direction: row
		justify-content: left
		align-items: center
		border-radius: $radius
		gap: 0.5rem

		&-author
			color: $accent-lighter
			font-weight: 600

		p
			white-space: nowrap
			overflow: hidden
			text-overflow: ellipsis

		p:nth-child(3)
			flex:1
			font-size: 0.8rem
			opacity: 0.5

		&:hover
			background: mix($primary, $invert, 90%)

		p
			margin: 0
			padding: 0

	.likes
		display: flex
		flex-direction: row
		gap: 0.5rem
		align-items: center
		padding: 0.5rem
		border-radius: $radius
		transition: all 0.3s $curve

		&:hover
			background: $hover-accent

		&:active
			background: $click-accent

	.views
		display: flex
		flex-direction: row
		gap: 0.5rem
		align-items: center
		padding: 0.5rem

	&:hover
		background-color: $secondary

	&:hover .comment-textfield
		background-color: $primary

	.like-thumb
		font-size: 1rem
		color: $text

	.like-container
		display: flex
		flex-direction: row
		align-items: center
		justify-content: center
		gap: 0.5rem

.play-icon
	font-size: 3rem
	line-height: 1.3
	&:active
		scale: 1

h2
	font-size: 1.5rem
	line-height: 1
	font-weight: 500
	margin: 0
	padding: 0
</style>
