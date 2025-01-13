<template>
	<div>
		<label for="" class="form-label">{{ label }}</label>
		<input
			:type="inputType"
			class="form-control"
			id=""
			:placeholder="placeholderTxt"
			v-model="localValue"
		/>
	</div>
</template>

<script setup lang="ts">
import { ref, watch, defineEmits } from 'vue';

// 接收 props
const props = defineProps<{
	label: string;
	placeholderTxt?: string;
	inputType?: string;
	modelValue: string | number; // 用來接收 v-model 的值
}>();

// 發送事件
const emit = defineEmits(['update:modelValue']);

// 使用本地變量來綁定值
const localValue = ref(props.modelValue);

// 監聽 props.modelValue 的變化，並同步到 localValue
watch(
	() => props.modelValue,
	newValue => {
		localValue.value = newValue;
	}
);

// 監聽 localValue 的變化，發送事件更新父組件
watch(
	() => localValue.value,
	newValue => {
		emit('update:modelValue', newValue);
	}
);
</script>

<style lang="scss" scoped></style>
