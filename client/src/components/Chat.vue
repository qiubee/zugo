<template>
  <div class="chat">
	<div class="chat_header">

	</div>
	<div class="chat_conversation empty" v-if="messages.length === 0">
		<p>Tijd voor een nieuwe reis en een gezellig gesprek!</p>
	</div>
	<div class="chat_conversation" v-else>
		<ul>
			<li v-for="message, index in messages" :key="index">
				<ChatMessage
				:key="index"
				:user="message.user"
				:message="message.message"
				:datetime="message.datetime"
				/>
			</li>
		</ul>
	</div>
	<div class="chat_footer">
		<form action="">
			<input type="text">
			<button class="button">Verstuur</button>
		</form>
	</div>
  </div>
</template>

<script>
import ChatMessage from "@/components/ChatMessage.vue";

export default {
	name: "Chat",
	components: {
		ChatMessage
	},
	props: {
		socket: Object,
		room: Object,
		user: Object
	},
	data() {
		return {
			messages: Array
		};
	},
	mounted() {
		const socket = this.socket;
		const vm = this;
		const room = this.room;
		const user = this.user;

		const form = document.querySelector(".chat_footer form");
		const input = document.querySelector(".chat_footer form input");
		form.addEventListener("submit", function (event) {
			event.preventDefault();
			if (input.value === "") {
				return;
			}
			const message = {
				roomId: room.id,
				roomName: room.journeyId,
				userId: user.id,
				message: input.value,
				datetime: new Date().toUTCString()
			};
			socket.emit("add message", message);
			input.value = "";
		});

		socket.on("messages", function (messages) {
			const sortedMessages = messages.map(function (message) {
				message.datetime = new Date(message.datetime).toLocaleString();
				if (message.user.id === user.id) {
					message.user.sender = true;
				}
				return message;
			}).sort(function (a, b) {
				return new Date(b.datetime) < new Date(a.datetime);
			});
			vm.messages = sortedMessages;
		});

		socket.on("new message", function (message) {
			if (message.user.id === user.id) {
				message.user.sender = true;
			}
			vm.messages.push(message);
		});
	}
};
</script>

<style scoped>
form {
	display: flex;
	flex-direction: row;
}

form input {
	max-width: 11.25rem;
}

form button {
	margin-left: 0.75rem;
}

.chat {
	height: 50vh;
}

.chat_conversation ul {
	display: flex;
	flex-direction: column;
	justify-content: left;
}

.chat_conversation li.sender {
	margin-left: auto;
}

.empty {
	position: relative;
	height: 45vh;
}

.empty p {
	text-align: center;
	font-weight: 500;
	font-size: 1.25rem;
	position: absolute;
	top: 50%;
	transform: translateY(-50%);
}

.button {
	font-weight: 500;
}
</style>