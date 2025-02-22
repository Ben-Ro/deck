<!--
  - @copyright Copyright (c) 2018 Julius Härtl <jus@bitgrid.net>
  -
  - @author Julius Härtl <jus@bitgrid.net>
  -
  - @license GNU AGPL version 3 or any later version
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<template>
	<div v-if="card" class="badges">
		<div v-if="card.commentsCount > 0"
			v-tooltip="commentsHint"
			class="icon icon-comment"
			:class="{ 'icon-comment--unread': card.commentsUnread > 0 }"
			@click.stop="openComments">
			{{ card.commentsCount }}
		</div>

		<div v-if="card.description && checkListCount > 0" class="card-tasks icon icon-checkmark">
			{{ checkListCheckedCount }}/{{ checkListCount }}
		</div>
		<div v-else-if="card.description.trim() && checkListCount == 0" class="icon icon-description" />

		<div v-if="card.attachmentCount > 0" class="icon-attach icon icon-attach-dark">
			{{ card.attachmentCount }}
		</div>

		<AvatarList :users="card.assignedUsers" />

		<CardMenu class="card-menu" :card="card" />
	</div>
</template>
<script>
import AvatarList from './AvatarList'
import CardMenu from './CardMenu'

export default {
	name: 'CardBadges',
	components: { AvatarList, CardMenu },
	props: {
		card: {
			type: Object,
			default: null,
		},
	},
	computed: {
		checkListCount() {
			return (this.card.description.match(/^\s*([*+-]|(\d\.))\s+\[\s*(\s|x)\s*\](.*)$/gim) || []).length
		},
		checkListCheckedCount() {
			return (this.card.description.match(/^\s*([*+-]|(\d\.))\s+\[\s*x\s*\](.*)$/gim) || []).length
		},
		commentsHint() {
			if (this.card.commentsUnread > 0) {
				return t('deck', '{count} comments, {unread} unread', {
					count: this.card.commentsCount,
					unread: this.card.commentsUnread,
				})
			}
			return null
		},
	},
	methods: {
		openComments() {
			const boardId = this.card && this.card.boardId ? this.card.boardId : this.$route.params.id
			this.$router.push({ name: 'card', params: { id: boardId, cardId: this.card.id, tabId: 'comments' } })
		},
	},
}
</script>

<style lang="scss" scoped>
	.badges {
		display: flex;
		width: 100%;
		flex-grow: 1;

		.icon {
			opacity: 0.5;
			padding: 10px 20px;
			padding-right: 4px;
			margin-right: 5px;
			background-position: left;
			background-size: 16px;
			span {
				margin-left: 18px;
			}
			&.icon-comment--unread {
				opacity: 1;
			}
		}
	}

	.badges .icon.due {
		background-position: 4px center;
		border-radius: 3px;
		margin-top: 10px;
		margin-bottom: 10px;
		padding: 4px;
		font-size: 90%;
		display: flex;
		align-items: center;
		opacity: .5;
		flex-shrink: 1;

		.icon {
			background-size: contain;
		}

		&.overdue {
			background-color: var(--color-error);
			color: var(--color-primary-text);
			opacity: .7;
		}
		&.now {
			background-color: var(--color-warning);
			opacity: .7;
		}
		&.next {
			background-color: var(--color-background-dark);
			opacity: .7;
		}

		span {
			margin-left: 20px;
			white-space: nowrap;
			text-overflow: ellipsis;
			overflow: hidden;
		}
	}

	.fade-enter-active, .fade-leave-active {
		transition: opacity .125s;
	}

	.fade-enter, .fade-leave-to {
		opacity: 0;
	}

	@media print {
		.badges {
			align-items: flex-start;
			max-height: none !important;
		}

		.card-menu {
			display: none;
		}
	}
</style>
