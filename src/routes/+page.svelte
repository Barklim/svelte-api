<script lang="ts">
	/**
	 * Dependence
	 */
	import { fade } from 'svelte/transition';
	import { getPosts } from '$lib/api/methods/posts';
	import { getReasonPhrase } from 'http-status-codes';
  import { onMount } from 'svelte';

	/**
	 * Components
	 */
  import Loader from '$lib/components/Loader.svelte';

  let posts = [];
  let loading = true;
  let errorMessage = "";
  let notification = "";

  const LOCAL_STORAGE_KEY = "cachedPosts";

  const loadData = async () => {
    try {
      const response = await getPosts();

      if (!response.success) {
        throw new Error(getReasonPhrase(response.data.code));
      }

      localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(response.data));

      return response.data;
    } catch (error) {
      const cachedPosts = localStorage.getItem(LOCAL_STORAGE_KEY);

      if (cachedPosts) {
        notification = "Данные загружены из кеша.";
        return JSON.parse(cachedPosts);
      }

      // If no data in cache or network
      throw new Error("Нет данных для отображения.");
    }
  };

  onMount(async () => {
    try {
      posts = await loadData();
    } catch (error) {
      errorMessage = error.message;
    } finally {
      loading = false;
    }
  });
</script>

<div class="layout">
	<h1>Latest posts</h1>

	{#if loading}
		<Loader />
  {:else if errorMessage}
    <h1>Ошибка: {errorMessage}</h1>
  {:else}
    {#if notification}
      <div class="notification">
        {notification}
      </div>
    {/if}
		<div class="posts">
			{#each posts as post (post.id)}
      	<div class="posts__item" in:fade>
					<h4>{post.title}</h4>
					<p>{post.body}</p>
				</div>
        {/each}
    </div>
  {/if}
</div>


<style lang="scss">
  .layout {
    padding: 25px 50px;

    display: flex;
    flex-direction: column;

    justify-content: center;
    align-items: center;

    gap: 50px;

    width: 100%;
    height: max-content;

    h1 {
      text-align: center;
    }
  }

  .posts {
    display: flex;
    flex-direction: column;
    gap: 5px;
    width: 500px;


    &__item {
      padding: 10px 15px;
      display: flex;
      flex-direction: column;
      gap: 15px;

      height: 150px;
      width: 100%;

      border-radius: 8px;

      background: rgba(0, 0, 0, 0.08);


      h4 {
        color: #2b302c;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }

      p {
        overflow: hidden;
        display: -webkit-box;
        -webkit-line-clamp: 2; /* number of lines to show */
        line-clamp: 2;
        -webkit-box-orient: vertical;

        color: rgba(43, 48, 44, 0.7);
      }
    }
  }
</style>
