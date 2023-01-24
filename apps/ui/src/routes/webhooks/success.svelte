<script lang="ts">
	import { page } from '$app/stores';
	const gitlabToken = $page.url.searchParams.get('token');
	const bitbucketToken = $page.url.hash.split('&')[0].split('=')[1];
	if (gitlabToken) {
		localStorage.setItem('gitLabToken', gitlabToken);
	}
	if (bitbucketToken) {
		localStorage.setItem('bitbucketToken', bitbucketToken);
	}
	window.close();
</script>
