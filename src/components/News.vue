<template>
	<div class="news">
		<a class="newsBlock" :href="newsStuff.newsUrl">
				<img id="newsIcon" :src="newsStuff.newsIconUrl">
				<div>
					<p id="newsTitle" :href="newsStuff.newsUrl">{{newsStuff.newsTitle}}</p>
				</div>
		</a>
	</div>
</template>

<script>
export default {
	name: 'news',
	props: {
		newsStuff: Object,
	}
}
</script>

<style scoped>
.news{
	display:flex;
	flex-direction: row;
	justify-content: center;
	margin:auto;
	border-color: rgba(0, 0, 0, .1);
	border-radius: 7px;
	border-width: 1px;
	padding: 10px;
	text-align: start;
}
.newsBlock{
	display: flex;
	flex-direction: row;
	align-items: center;
	margin:auto;
}
#newsTitle{
	font-size: 0.9em;
	text-decoration: underline;
	margin-left: 15px;
}
#newsIcon{
	max-height: 9vh;
	max-width: 16vw;
	min-height: 6vh;
	min-width: 7vw;
	border-radius: 5px
}
</style>