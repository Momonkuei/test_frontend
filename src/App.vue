<template>
	<div class="container d-flex flex-wrap">
		<div class="col-12 col-md-6 p-3">
			<div class="border rounded p-3">
				<h2>操作</h2>
				<form @submit.prevent>
					<BFormControls
						class="mb-3"
						label="名字"
						v-model="formDate.name"
						placeholderTxt="請輸入名子"
						inputType="text"
					/>

					<BFormControls
						class="mb-3"
						label="年齡"
						type="number"
						v-model="formDate.age"
						placeholderTxt="請輸入年齡"
						inputType="number"
					/>

					<div class="d-flex gap-1">
						<button class="btn btn-success" @click="edit">
							修改
						</button>
						<button class="btn btn-warning" @click="create">
							新增
						</button>
					</div>
				</form>
			</div>
		</div>

		<div class="col-12 col-md-6 p-3">
			<div class="border rounded p-3">
				<div class="spin" v-if="!users.length || isLoading"></div>
				<table class="table" v-else>
					<thead>
						<tr>
							<th scope="col">#</th>
							<th scope="col">名字</th>
							<th scope="col">年齡</th>
							<th scope="col">操作</th>
						</tr>
					</thead>
					<tbody>
						<tr v-for="(v, i) in users" :key="`user_${i}`">
							<th scope="row">{{ v.id }}</th>
							<td>{{ v.name }}</td>
							<td>{{ v.age }}</td>
							<td class="d-flex gap-1">
								<button
									class="btn btn-success"
									@click="selectUser(v)"
								>
									修改
								</button>
								<button
									class="btn btn-danger"
									@click="remove(v)"
								>
									刪除
								</button>
							</td>
						</tr>
					</tbody>
				</table>
			</div>
		</div>
	</div>
</template>

<script setup lang="ts">
import axios from 'axios';
import { onMounted, ref, computed } from 'vue';
import BFormControls from './components/BFormControls.vue';

interface User {
	id: number;
	name: string;
	age: number;
}

const baseUrl = 'https://ijk7bd8l.wuc.us.kg'; // 由面試官提供
const users = ref<User[]>([]);
const formDate = ref({
	// id readonly
	id: 0,
	name: '',
	age: 0,
});

const isLoading = ref(false);
const nameIsEmpty = computed(() => formDate.value.name.trim() === '');
const ageIsInvalid = computed(() => Number(formDate.value.age) <= 0);
const fieldsRules = computed(() => nameIsEmpty.value || ageIsInvalid.value);

const create = () => {
	// 需有確認步驟

	if (fieldsRules.value) {
		console.log('名字或是年齡未填');
		alert('名字或是年齡未填');
		return;
	} else {
		// users 最後一個id 號碼再加1做為新增用戶id
		// formDate.value.id = (
		// 	parseInt(users.value[users.value.length - 1].id) + 1
		// ).toString();

		// 因為id 不是自己添加，只好請求了
		// users.value.push({ ...formDate.value });

		isLoading.value = true;

		axios({
			method: 'post',
			url: baseUrl + '/api/user',
			data: {
				age: formDate.value.age,
				name: formDate.value.name,
			},
		})
			.then(function (res) {
				console.log(res);
				isLoading.value = false;
				getUsers();
				formDate.value.name = '';
				formDate.value.age = 0;
			})
			.catch(function (err) {
				console.log(err);
			});
	}
};

// 更新資訊
const updateUser = () => {
	const index = users.value.findIndex(u => u.id === formDate.value.id);
	if (index !== -1) {
		users.value[index] = { ...formDate.value }; // 使用淺拷貝更新數據
	}
};

const edit = () => {
	// 需有確認步驟

	if (fieldsRules.value) {
		console.log('名字或是年齡未填');
		alert('名字或是年齡未填');
		return;
	} else {
		updateUser();

		let checkedEdit = confirm(`確認修改${formDate.value.name}的資料嗎?`);

		if (!checkedEdit) {
			return;
		} else {
			// console.log(formDate.value);
			isLoading.value = true;

			axios({
				method: 'put',
				url: baseUrl + '/api/user',
				data: formDate.value,
			})
				.then(() => {
					console.log('修改成功');
					isLoading.value = false;

					formDate.value.name = '';
					formDate.value.age = 0;
				})
				.catch(err => {
					console.log(err);
				});
		}
	}
};

const selectUser = (user: User) => {
	// 禁止使用 formDate.value = user
	formDate.value = { ...user };
	// console.log(formDate.value);
};

const remove = (user: User) => {
	// 需有確認步驟
	let checkedRemove = confirm(`確認刪除${user.name}的資料嗎?`);
	if (!checkedRemove) {
		return;
	} else {
		isLoading.value = true;
		axios({
			method: 'delete',
			url: baseUrl + '/api/user',
			data: {
				id: user.id,
			},
		})
			.then(res => {
				isLoading.value = false;
				console.log(res.data);
				users.value = users.value.filter(
					dataUser => dataUser.id !== user.id
				);
			})
			.catch(function (err) {
				console.log(err);
			});
	}
};

const getUsers = () => {
	axios({
		method: 'get',
		url: baseUrl + '/api/user',
	})
		.then(res => {
			users.value = res.data.data;
		})
		.catch(err => {
			console.log(err);
		});
};

const setupPage = () => {
	getUsers();
};

onMounted(setupPage);
</script>

<style scoped>
.spin {
	display: block;
	width: 40px;
	height: 40px;
	margin: 30px auto;
	border: 3px solid transparent;
	border-radius: 50%;
	border-top-color: #ff8800;
	animation: spin 1s ease infinite;
}

@keyframes spin {
	to {
		-webkit-transform: rotateZ(360deg);
	}
}
</style>
