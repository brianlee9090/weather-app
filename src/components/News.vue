<template>
	<div class="news">
		<div class="newsBlock">
				<img id="newsIcon" :src="newsStuff.newsIconUrl">
				<div>
					<a id="newsTitle" :href="newsStuff.newsUrl">{{newsStuff.newsTitle}}</a>
				</div>
		</div>
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
	background-color: honeydew;
	border-radius: 15px;
	padding: 10px;
}
.newsBlock{
	display: flex;
	flex-direction: row;
	align-items: center;
	margin:auto;
}
#newsTitle{
	font-size:min(2vh, 11px);
	text-decoration: underline;
	margin-left: 5px
}
#newsIcon{
	max-height: 9vh;
	max-width: 16vw;
	border-radius: 5px
}
</style>