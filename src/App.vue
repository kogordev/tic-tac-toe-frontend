<script setup>
import { ref, computed, onMounted } from 'vue';
import { io } from 'socket.io-client';

const ENDPOINT = 'ws://localhost:3000';
const socket = io(ENDPOINT, { transports: ['websocket'] });

socket.on('draw', (data) => {
	console.log(data);
	MakeMove(...data, true);
});

socket.on('gameReset', (data) => {
	console.log('gameReset recieved');
	GameReset(true);
});

onMounted(() => {
	console.log('refreshed');
	GameReset(false);
});

const board = ref([
	['', '', ''],
	['', '', ''],
	['', '', ''],
]);

const boardIsNotFull = ref(null);
const player = ref('X');

const CalculateWinner = (squares) => {
	const lines = [
		[0, 1, 2],
		[3, 4, 5],
		[6, 7, 8],
		[0, 3, 6],
		[1, 4, 7],
		[2, 5, 8],
		[0, 4, 8],
		[2, 4, 6],
	];

	for (let i = 0; i < lines.length; i++) {
		let [a, b, c] = lines[i];
		if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
			return squares[a];
		}
	}
	boardIsNotFull.value = squares.includes('');
	return null;
};

const winner = computed(() => CalculateWinner(board.value.flat()));

const MakeMove = (x, y, serverAction) => {
	if (board.value[x][y] || winner.value) {
		return;
	}
	board.value[x][y] = player.value;
	if (!serverAction) {
		socket.emit('move', [x, y]);
	}
	player.value = player.value === 'X' ? 'O' : 'X';
};

const GameReset = (serverAction) => {
	board.value = [
		['', '', ''],
		['', '', ''],
		['', '', ''],
	];

	player.value = 'X';
	if (!serverAction) {
		socket.emit('reset', '');
		console.log('reset emited');
	}
};
</script>

<template>
	<main class="flex flex-col items-center justify-center pt-8">
		<h1 class="uppercase text-4xl font-bold mb-8">tic tac toe</h1>
		<Transition name="bounce" mode="out-in">
			<h2 class="text-xl font-bold mb-4npm" v-if="!winner && boardIsNotFull">
				Player {{ player }}'s turn
			</h2>
			<h2
				class="text-xl font-bold mb-4 border rounded bg-green-500 text-white px-2 py-1"
				v-else-if="winner"
			>
				Player {{ winner }}'s WINS!
			</h2>
			<h2
				class="text-xl font-bold mb-4 border rounded bg-blue-500 text-white px-2 py-1"
				v-else
			>
				DRAW!
			</h2>
		</Transition>
		<div class="flex flex-col items-center mb-8 text-black">
			<div class="flex" v-for="(row, x) in board" :key="x">
				<div
					:class="`flex items-center justify-center border hover:bg-slate-300 cursor-pointer h-20 w-20  font-medium ${
						cell === 'X' ? 'text-pink-500' : 'text-blue-500'
					} duration-100`"
					v-for="(cell, y) in row"
					:key="y"
					@click="MakeMove(x, y, false)"
				>
					{{ cell }}
				</div>
			</div>
		</div>
		<button
			class="text-xl font-bold bg-pink-500 text-white py-1 px-2 rounded hover:bg-pink-600 duration-300"
			@click="GameReset(false)"
		>
			Reset
		</button>
	</main>
</template>

<style scoped>
/*
  Enter and leave animations can use different
  durations and timing functions.
*/
.bounce-enter-active {
	animation: bounce-in 0.1s;
}
.bounce-leave-active {
	animation: bounce-in 0.1s reverse;
}
@keyframes bounce-in {
	0% {
		transform: scale(0);
	}
	50% {
		transform: scale(1.25);
	}
	100% {
		transform: scale(1);
	}
}
</style>
