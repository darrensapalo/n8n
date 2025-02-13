<template>
	<div :class="$style.container" v-show="loading || collections.length">
		<agile
			ref="slider"
			:dots="false"
			:navButtons="false"
			:infinite="false"
			:slides-to-show="4"
			@after-change="updateCarouselScroll"
		>
			<Card v-for="n in loading ? 4 : 0" :key="`loading-${n}`" :loading="loading" />
			<TemplatesInfoCard
				v-for="collection in loading ? [] : collections"
				data-test-id="templates-info-card"
				:key="collection.id"
				:collection="collection"
				:showItemCount="showItemCount"
				:width="cardsWidth"
				@click="(e) => onCardClick(e, collection.id)"
			/>
		</agile>
		<button
			v-show="showNavigation && carouselScrollPosition > 0"
			:class="{ [$style.leftButton]: true }"
			@click="scrollLeft"
		>
			<font-awesome-icon icon="chevron-left" />
		</button>
		<button
			v-show="showNavigation && !scrollEnd"
			:class="{ [$style.rightButton]: true }"
			@click="scrollRight"
		>
			<font-awesome-icon icon="chevron-right" />
		</button>
	</div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import type { PropType } from 'vue';
import type { ITemplatesCollection } from '@/Interface';
import Card from '@/components/CollectionWorkflowCard.vue';
import TemplatesInfoCard from '@/components/TemplatesInfoCard.vue';
import { VueAgile } from 'vue-agile';

import { genericHelpers } from '@/mixins/genericHelpers';

type SliderRef = InstanceType<typeof VueAgile>;

export default defineComponent({
	name: 'TemplatesInfoCarousel',
	mixins: [genericHelpers],
	props: {
		collections: {
			type: Array as PropType<ITemplatesCollection[]>,
		},
		loading: {
			type: Boolean,
		},
		showItemCount: {
			type: Boolean,
			default: true,
		},
		showNavigation: {
			type: Boolean,
			default: true,
		},
		cardsWidth: {
			type: String,
			default: '240px',
		},
	},
	watch: {
		collections() {
			setTimeout(() => {
				this.updateCarouselScroll();
			}, 0);
		},
		loading() {
			setTimeout(() => {
				this.updateCarouselScroll();
			}, 0);
		},
	},
	components: {
		Card,
		TemplatesInfoCard,
		agile: VueAgile,
	},
	data() {
		return {
			carouselScrollPosition: 0,
			cardWidth: parseInt(this.cardsWidth, 10),
			sliderWidth: 0,
			scrollEnd: false,
			listElement: null as null | Element,
		};
	},
	methods: {
		updateCarouselScroll() {
			if (this.listElement) {
				this.carouselScrollPosition = Number(this.listElement.scrollLeft.toFixed());

				const width = this.listElement.clientWidth;
				const scrollWidth = this.listElement.scrollWidth;
				const scrollLeft = this.carouselScrollPosition;
				this.scrollEnd = scrollWidth - width <= scrollLeft + 7;
			}
		},
		onCardClick(event: MouseEvent, id: string) {
			this.$emit('openCollection', { event, id });
		},
		scrollLeft() {
			if (this.listElement) {
				this.listElement.scrollBy({ left: -(this.cardWidth * 2), top: 0, behavior: 'smooth' });
			}
		},
		scrollRight() {
			if (this.listElement) {
				this.listElement.scrollBy({ left: this.cardWidth * 2, top: 0, behavior: 'smooth' });
			}
		},
	},
	async mounted() {
		await this.$nextTick();
		const sliderRef = this.$refs.slider as SliderRef | undefined;
		if (!sliderRef) {
			return;
		}

		this.listElement = sliderRef.$el.querySelector('.agile__list');
		if (this.listElement) {
			this.listElement.addEventListener('scroll', this.updateCarouselScroll);
		}
	},
	beforeUnmount() {
		const sliderRef = this.$refs.slider as SliderRef | undefined;
		if (sliderRef) {
			sliderRef.destroy();
		}

		window.removeEventListener('scroll', this.updateCarouselScroll);
	},
});
</script>

<style lang="scss" module>
.container {
	position: relative;
}

.button {
	width: 28px;
	height: 37px;
	position: absolute;
	top: 35%;
	border-radius: var(--border-radius-large);
	border: var(--border-base);
	background-color: #fbfcfe;
	cursor: pointer;

	&:after {
		content: '';
		width: 40px;
		height: 140px;
		top: -55px;
		position: absolute;
	}
	svg {
		color: var(--color-foreground-xdark);
	}
}

.leftButton {
	composes: button;
	left: -30px;

	&:after {
		left: 27px;
		background: linear-gradient(
			270deg,
			hsla(
				var(--color-background-light-h),
				var(--color-background-light-s),
				var(--color-background-light-l),
				50%
			),
			hsla(
				var(--color-background-light-h),
				var(--color-background-light-s),
				var(--color-background-light-l),
				100%
			)
		);
	}
}

.rightButton {
	composes: button;
	right: -30px;

	&:after {
		right: 27px;
		background: linear-gradient(
			90deg,
			hsla(
				var(--color-background-light-h),
				var(--color-background-light-s),
				var(--color-background-light-l),
				50%
			),
			hsla(
				var(--color-background-light-h),
				var(--color-background-light-s),
				var(--color-background-light-l),
				100%
			)
		);
	}
}
</style>

<style lang="scss">
.agile {
	&__list {
		width: 100%;
		padding-bottom: var(--spacing-2xs);
		overflow-x: auto;
		transition: all 1s ease-in-out;
	}
}
</style>
