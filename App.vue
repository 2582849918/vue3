<template>
	<div class="request flex flex-col">
		<v-header title="Request For Testing" :type="0" @request="request_test" />
		<div class="content flex flex-col">
			<!-- head布局 -->
			<div class="request-head">
				<div class="image-head">
					<router-link h-60 f-c-c to="/">
						<!-- <img
							class="user-img img-circle"
							src="../Home/mts.png"
							alt=" "
							style="height: 40px; width: 70px"
						/> -->
					</router-link>
				</div>
				<div class="text-container">
					<h3 class="text-center" style="font-size: 18px; margin-top: 20px">
						MTS Request For Testing
					</h3>
				</div>
				<div class="Help">
					<n-button @click="show = true">Help Guide</n-button>
					<n-drawer v-model:show="show" :width="502">
						<n-drawer-content title="Help Guide" closable>
							<n-collapse>
								<n-collapse-item
									title="Device Info"
									name="1"
									style="font-size: 20px"
								>
									<div>&nbsp; &nbsp; Device Status Description</div>
									<n-gradient-text size="15px">
										&nbsp; &nbsp; &nbsp; 1. Green indicates the device is in
										idle state. </n-gradient-text
									><br />
									<n-gradient-text size="15px">
										&nbsp; &nbsp; &nbsp; 2. Gray means the device is offline.
									</n-gradient-text>
									<n-gradient-text size="15px">
										&nbsp; &nbsp; &nbsp; 3. Red means the device is running.
									</n-gradient-text>
								</n-collapse-item>
								<n-collapse-item
									title="Load Path"
									name="2"
									style="font-size: 15px"
								>
									<div>
										&nbsp; &nbsp; &nbsp; Enter the Load path to be tested in
										Load Path. Leaving it blank means the default selection of
										Load from the device.
									</div>
								</n-collapse-item>
								<n-collapse-item
									title="Test Plan"
									name="3"
									style="font-size: 15px"
								>
									<div>
										&nbsp; &nbsp; &nbsp; 1. In the Test Plan, you can opt to
										test according to Domain or some test cases within the
										Domain (select up to 30 test cases at most).<br />
										&nbsp; &nbsp; &nbsp; 2. When you need to recreate a
										scenario, you can compare the time of DB Dump and the time
										of running the test cases on the Log, then select a few or
										multiple test cases around the time of Dump DB for
										retesting. <br />
										&nbsp; &nbsp; &nbsp; 3. After the test is finished, you can
										view the test report and download the Log from the Test
										Report interface.
										<a
											href="http://mts.gcn.mediatek.inc/system"
											style="color: #18a058; text-decoration: underline"
											>Test Report</a
										>.
									</div>
								</n-collapse-item>
								<n-collapse-item
									title="Download Case"
									name="4"
									style="font-size: 15px"
								>
									<div>
										&nbsp; &nbsp; &nbsp; It supports downloading cases to local
										for testing, because for long-term Domain, it is necessary
										to download testing resource files to local first.<a
											href="https://share.mediatek.inc/browse/shares/69? path=%2FMTS_Download_Source&sort=type#"
											style="color: #18a058; text-decoration: underline"
											>Download Source</a
										>
									</div>
								</n-collapse-item>
							</n-collapse>
						</n-drawer-content>
					</n-drawer>
				</div>
			</div>
			<div style="margin: 0 auto; width: 80%">
				<div class="tab container">
					<n-tabs
						type="card"
						placement="top"
						size="large"
						animated
						v-model:value="currentTab"
						@update:value="tabClick"
					>
						<n-tab-pane
							v-for="item in tabs"
							:key="item.tit"
							:name="item.tit"
							:tab="item.tit"
						>
							<n-checkbox-group
								:value="checkbox"
								@update:value="handleUpdateValue"
							>
								<n-grid :y-gap="8" :cols="4">
									<n-gi v-for="subltem in item.checkList" :key="subltem.name">
										<n-checkbox
											:value="subltem.name"
											:label="subltem.name"
											:disabled="subltem.status == 'grey'"
										/>
										<span class="status-icon" :class="subltem.status"></span>
									</n-gi>
								</n-grid>
							</n-checkbox-group>
							<div v-if="currentTab === 'Customer'" />
							<div v-if="currentTab === 'Migration Chip'" />
						</n-tab-pane>
					</n-tabs>
				</div>
			</div>
			<div class="list_request flex-col flex">
				<div class="search">
					<div class="tit">
						<div class="input flex" style="justify-content: flex-end">
							<span>* Load Path: </span>
							<!-- <n-input v-model:value="formValue.desc" placeholder="EX. 公共盤路徑：\\alpsfs.mediatek.inc|sw2oa|user\ot904775\20231018202757"> -->
						</div>
					</div>
				</div>
				<div class="table" style="max-height: 500px; overflow: auto">
					<div class="table-header flex y-center">
						<div class="table-header-row">Test Plan</div>
						<div class="table-header-row">Description</div>
					</div>
					<div
						class="row flex flex-col x-center"
						v-for="(row, idx) in data"
						@click="handleExpandChange(row, idx)"
						:key="row.type_case_id"
					>
						<div :class="['row-header flex', { border: row.show }]">
							<div class="flex-center left">
								<el-popover
									placement="right-start"
									:width="200"
									trigger="hover"
									:content="row.name"
								>
									<template #reference>
										<div class="flex-center">
											<el-icon v-if="!row.show"><ArrowRight /></el-icon>
											<el-icon v-else><ArrowDown /></el-icon>
											<div class="tit">{{ row.name }}</div>
										</div>
									</template>
								</el-popover>
							</div>
							<div class="right flex y-center">
								<el-button
									type="text"
									class="your-custom-class"
									@click.stop="getRowData(row)"
									>Detail</el-button
								>
							</div>
						</div>
						<div class="row-body" v-if="row.show" @click.stop>
							<el-table
								ref="multipleTable"
								row-key="id"
								:data="row.children"
								@select-all="
									(selectedRows) => handleSelectAll(row, selectedRows)
								"
								@select="(selectedRows) => handleSelect(row, selectedRows)"
							>
								<el-table-column type="selection" />
								<el-table-column prop="name" label="All Cases In Module" />
							</el-table>
						</div>
						<!-- </el-collapse-item> -->
						<!-- </el-collapse> -->
					</div>
					<!-- <el-table :data="data" tree-props="{children: 'children'}">
						<el-table-column type="expand">
							<template #default="{ row }">
								<el-table
									:data="row.children"
									@select-all="handleSelectAll"
									@select="handleSelect"
									@selection-change="
										(selectedRows) =>
											handleChildSelectDelayed(row, selectedRows)
									"
								>
									<el-table-column type="selection" />
									<el-table-column prop="name" label="All Cases In Module" />
								</el-table>
							</template>
						</el-table-column>
						<el-table-column pron="module name" label="Test Plan" />
						<el-table-column label="Description" fixed="right">
							<template #default="{ row }">
								<el-button
									type="text"
									class="your-custom-class"
									@click="getRowData(row)"
									>Detail</el-button
								>
							</template>
						</el-table-column>
					</el-table> -->
				</div>
				<div class="pagination-controls">
					<n-button type="primary" @click="apply" :disabled="!isFormValid"
						>Submit</n-button
					>
				</div>
			</div>
		</div>
		<n-modal
			style="
				width: 70%;
				position: fixed;
				top: 30%;
				left: 50%;
				transform: translateX(-50%);
			"
			v-model:show="showModal"
			preset="card"
			title="Detail"
			@positive-click="submitCallback"
			@negative-click="cancelCallback"
		>
			<p v-for="(item, index) in contents" :key="index">
				{{ selectedOperation }}
			</p>
		</n-modal>
	</div>
	<!-- <div v-html="content"></div> -->
</template>

<script setup>
	import { ElMessage } from 'element-plus';

	import { h, reactive, ref, onMounted, computed, nextTick } from 'vue';
	// import { NButton, useDialog, useMessage, NTooltip } from 'naive-ui';
	import { ElTable, ElTableColumn } from 'element-plus';
	// import 'element-plus/dist/ index.css';
	import axios from 'axios';
	const show = ref(false);
	const token = ref(null);
	// token.value = getSharedValue();

	const multipleTable = ref([]);

	const handleSelect = (parentRow, selectedRows) => {
		console.log('handleSelect::: ', parentRow, selectedRows);
		// 选的时候，先将所有 isCheck 重置，防止选中之后取消了，没有给之前选中的状态清空，导致下次打开又勾上了
		parentRow.children.forEach((row) => {
			row.isCheck = false;
		});
		// 将选中的 isCheck = true ，用于打开时候查找为 true 的去设置选中
		selectedRows.forEach((row) => {
			row.isCheck = true;
		});
		if (selectedRows.length > 50) {
			// canSubmit.value = 0;
			ElMessage.warning('this is a message.');
			// console.warn("Maximum of 50 rows can be selected");
		}
		// const currentSelectedCount = Array.from(
		//   selectedChildren.value.values()
		// ).flat().length;
		//当手动选择或者取消选择时，也同样更新isSelected.value
		// isSelected.value = currentSelectedCount !== 0;
	};

	//
	const handleSelectAll = (parentRow, selectedRows) => {
		console.log('handleSelectAll::: ', parentRow, selectedRows);

		// selectedRows 取消全选，就给所有设置 isCheck false
		if (!selectedRows.length) {
			parentRow.children.forEach((item) => {
				item.isCheck = false;
			});
		} else {
			// selectedRows 全选，就给所有设置 isCheck true
			selectedRows.forEach((item) => {
				item.isCheck = true;
			});
		}
		// 全选操作，不需要检查选择的数量
		isSelected.value = true;
	};

	// 菜单触发事件
	const handleExpandChange = (row, idx) => {
		row.show = !row.show;
		if (!row.show) return;

		// 表格渲染需要一段时间 nextTick 能确保 table 已经成功渲染
		nextTick(() => {
			console.log(
				'handleExpandChange::: ',
				row,
				multipleTable.value,
				multipleTable.value.length,
				multipleTable.value[idx],
				idx
			);
			row.children.forEach((item) => {
				// isCheck 为 true 的进行设置
				if (item.isCheck) {
					// 这里设置选中，通过ref拿到table的实例，
					// 调用 toggleRowSelection 去进行设置选择
					// 为什么要 multipleTable?.value.length - 1 应该表格每打开一次都会追加一个实例，这样能保证是给最后一个table设置选中
					multipleTable?.value[
						multipleTable?.value.length - 1
					].toggleRowSelection(item, item.isCheck);
				}
			});
		});
		//   // }
		// });
	};

	//在 script setup 里增加
	const tabs = ref([]);
	// 我们会在下面填充这个数据
	// async function fetchChecklistData() {
	//     try {
	//         const response = await axios.get ('/api/chips_status');
	//         if (response && response.data)
	//         {
	//             const rawData = response.data;
	//             console.log(rawData);
	//             // 将数据转化为我们需要的格式
	//             const formattedData = rawData.reduce ((acc, cur) => {
	//                 const existing Tab = acc.find(item => item.tit === cur.type);
	//                 const statusColor = cur.status === 'used' ? 'red' : (cur.status === 'not connected'? 'grey' : 'green');
	//                 if (existing Tab) {
	//                     existing Tab.checkList.push({ name:cur.name, status: statusColor });
	//                 } else {
	//                     acc.push({
	//                         tit: cur.type,
	//                         key: cur.type,
	//                         checkList: [{ name: cur.name, status: statusColor }],
	//                         disabled: false
	//                     });
	//                 }

	//             return acc;
	//             },[]);
	//             tabs.value = formattedData;
	//             console.log (tabs.value);
	//         }
	//     } catch (error) {
	//     console.error (error);
	//     }
	// }

	const currentTab = ref('');
	onMounted(async () => {
		// await fetchChecklistData();
		// if (tabs.value && tabs.value.length > 0) {
		// 	currentTab.value = tabs.value[0].tit;
		// }
	});

	// 表格数据
	const data = ref([
		{
			children: Array.from({ length: 100 }).map((_, index) => ({
				name: index,
				id: Date.now() + Math.random() + index + 8,
				isCheck: false,
			})),
			show: false,
			name: 'Audio',
			operation: 'xxxxx',
			type_case_id: 1,
		},
		{
			children: Array.from({ length: 60 }).map((_, index) => ({
				name: index,
				id: Date.now() + Math.random() + index + 8,
				isCheck: false,
			})),
			show: false,
			name: 'Audio',
			operation: 'xxxxx',
			type_case_id: 3,
		},
		{
			children: [
				{
					name: '001nnn',
					id: Date.now() + Math.random() + 3,

					isCheck: false,
				},
				{
					name: '002nnn',
					id: Date.now() + Math.random() + 4,

					isCheck: false,
				},
				{
					name: '039nnn',
					id: Date.now() + Math.random() + 5,

					isCheck: false,
				},
			],
			show: false,
			name: 'Display',
			operation: 'nnnnn',
			type_case_id: 2,
		},
	]);
	const checkbox = ref([]);
	const checkedRowKeysRef = ref([]);
	// const dialog = useDialog();
	const canSubmit = ref(true);

	onMounted(async () => {
		try {
			const response = await axios.post('./api/get-domain-data');
			if (response && response.data) {
				const rawData = response.data; //假设 API 返回一个数据数组
				data.value.forEach((item) => {
					console.log(
						`children array for ${item.module_name}: `,
						item.children
					);
				});
				data.value = rawData.map((item) => {
					console.log(
						`children array for ${item.module_name} from API response: `,
						item.children
					);
					const newltem = {
						...item,
						children: Array.isArray(item.children) ? item.children : [],
					};
					// console.log( children array for ${item.module_name} after processing:', newltem.children);
					return newltem;
				});
			}
		} catch (e) {
			// handle the error
			console.error(e);
		}
	});

	const contents = ref(['内容1']);

	const tabClick = (tabName) => {
		currentlab.value = tabName;
	};
	const selectedChildren = ref(new Map());
	const isSelected = ref(false);
	// const handleSelectAll = () => {
	// 	// 全选操作，不需要检查选择的数量
	// 	isSelected.value = true;
	// };

	// const handleSelect = () => {
	// 	//当手动选择或者取消选择时，也同样更新isSelected.value
	// 	const currentSelectedCount = Array.from(
	// 		selectedChildren.value.values()
	// 	).flat().length;
	// 	isSelected.value = currentSelectedCount !== 0;
	// };

	// const handleChildSelectDelayed = (parentRow, selectedRows) => {
	// 	setTimeout(() => {
	// 		//将 selectedRows 存入selectedChildren
	// 		selectedChildren.value.set(
	// 			parentRow,
	// 			Array.isArray(selectedRows) ? selectedRows : []
	// 		);
	// 		let currentSelectedCount = 0;
	// 		// 遍历每个模块的子节点
	// 		for (let [parent, childRows] of selectedChildren.value.entries()) {
	// 			// 如果'All Cases In Module"被全选，那么不计入计数
	// 			if (parent.children.length !== childRows.length) {
	// 				// 如果'All Cases In Module'被部分选中，那么计算被选中的子节点数量
	// 				currentSelectedCount += childRows.length;
	// 				canSubmit.value = currentSelectedCount;
	// 			}
	// 		}
	// 		//如果选择数已经超过 50，则打印警告
	// 		if (currentSelectedCount > 50) {
	// 			canSubmit.value = 0;
	// 			message.warning('Maximum of 50 rows can be selected');
	// 		}
	// 		//如果全不选或者部分选中，将isSelected.value设置为 false
	// 		isSelected.value = currentSelectedCount !== 0;
	// 	}, 0);
	// };

	// const isFormValid = computed(() => Boolean( formValue.desc && selectedChildren.value.size && canSubmit.value));
	// const apply = async () => {
	//     console.error('formValue.desc.length', formValue.desc.length);
	//     //检查 formValue 的值
	//     if (!formValue.desc || formValue.desc.length === 0)
	//         return message.info("Please fill in the Load Path");

	//     //检查 selectedRows 的值
	//     if (selectedChildren.value.size === 0)
	//         return message.info ("Please select Test Plan");
	//     dialog.success({
	//         title: "System Prompt",
	//         content:"Confirm whether to submit the application for testing",
	//         positiveText:"Submit",
	//         negativeText:"Cancel",
	//         onPositiveClick: async () => {
	//             const info222 = {
	//                 token: token.value,
	//                 checkbox: checkbox.value,
	//                 formValue: formValue.desc,
	//                 selectedRows:Array.from(selectedChildren.value.entries()).map(([parent, children]) =>({
	//                     parent: parent.module_name, //获取父点的 module_name 属性
	//                     children: children.map(child => child.name)
	//                 }))
	//             };

	//         try {
	//             console.log(info222);
	//             const response = await fetch("/api/apply-test",{
	//                 method: 'POST',
	//                 headers: { 'Content-Type:'application/json' },
	//                 body:JSON.stringify(info222)， //使用JSON.stringify 转换info222为JSON字符串
	//             });
	//             const data = await response.json();
	//             console.error("data", data);
	//             if (data.message && data.message == 'DeviceConnected') {
	//                 window.history.back();
	//             } else if (data.message == 'Error: Device mismatch') {
	//                 message.error ("Error: Device mismatch");
	//             } else if (data.message =='Device Not Connected') {
	//                 message.warning ("Error: Device Not Connected");
	//             } else {
	//                 message.warning ("Error: This machine is currently working");
	//             }
	//         } catch(error) {
	//             message.warning('Error: Server Not Responding.');
	//         }
	//     },
	//     onNegativeClick: () => {
	//         message.error ("Unsuccessful");
	//     },
	//     });
	// };

	const showModal = ref(false);
	const selectedOperation = ref('');
	const getRowData = (rowData) => {
		selectedOperation.value = rowData.operation;
		showModal.value = true;
		console.log('rowData::: ', selectedOperation.value);
	};

	const submitCallback = () => {};
	const cancelCallback = () => {};

	const pagination = reactive({
		page: 1,
		pageSize: 7,
		onChange: (page) => {
			pagination.page = page;
		},
		onUpdatePageSize: (pageSize) => {
			pagination.pageSize = pageSize;
			pagination.page = 1;
		},
	});

	const handleUpdateValue = (value) => {
		console.log('value::: ', value);
		checkbox.value = value;
	};

	// const message = useMessage();
	// function request_test() {
	// 	if (!checkedRowKeysRef.value.length)
	// 		return message.warning('Please select data');
	// }

	const formValue = reactive({
		desc: '',
	});
</script>

<style lang="scss" scoped>
	::placeholder {
		color: rgba(0, 0, 0, 0.5);
	}

	.el-table-column {
		max-height: 300px;
		overflow-y: auto;
	}

	.status-icon {
		display: inline-block;
		width: 10px;
		height: 10px;
		margin-left: 5px;
	}

	.status-icon.green {
		background-color: green;
	}
	.status-icon.grey {
		background-color: grey;
	}
	.status-icon.red {
		background-color: red;
	}

	.request-head {
		margin-top: 10px;
		display: flex;
		justify-content: center;
		width: 80%;
		margin: 0 auto;
	}

	.pagination-controls {
		display: flex;
		align-items: center;
		justify-content: flex-end;
	}

	image-head {
		display: flex;
		justify-content: flex-start;
		align-items: center;
	}

	.text-container {
		width: 80%;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.user-img {
		height: 40px;
		width: 70px;
	}

	.your-custom-class {
		color: blue;
		text-decoration: underline;
	}

	.text-center {
		font-size: 18px;
		margin-top: 20px;
		text-align: center;
		flex: 1;
	}

	.Help {
		font-size: 18px;
		margin-top: 20px;
		text-align: center;
		flex: 1;
	}

	.request {
		height: 100%;
		overflow: auto;
		header {
			// background-color: #e9edfo;
		}
		.custom-input {
			width: 800px;
			font-size: 20px;
		}
		.content {
			flex: 1;
			background-color: #f6f6f6;
			.tab {
				margin-top: 10px;
				:deep(.n-checkbox-group) {
					margin-bottom: 10px;
				}
				:deep(.n-tab-pane) {
					margin-bottom: 10px;
				}
				:deep(.n-tabs-nav-scroll-content) {
					border-bottom: 2px solid #18a058;
				}
			}

			.list_request {
				width: 80%;
				margin: 0 auto 30px;
				flex: 1;
				.search {
					.tit {
						font-size: 18px;
						border-top: 2px solid #18a058;
						padding-top: 15px;
					}
					.input {
						margin: 20px 0;
						span {
							width: 15%;
						}
					}
				}

				.table {
					&-header {
						height: 50px;
						padding: 0 20px;
						background-color: white;
						border-bottom: 1px solid #ccc;

						&-row {
							width: 50%;
						}
					}
					.row {
						position: relative;
						min-height: 50px;
						background-color: white;
						border-bottom: 1px solid #ccc;
						&:last-child {
							border: none;
						}
						&-header {
							// position: fixed;
							// border-bottom: 1px solid #ccc;
							height: 50px;
							// width: 100%;
							padding: 0 20px;
							&.border {
								border-bottom: 1px solid #ccc;
							}
							.tit {
								margin-left: 20px;
							}
							.left {
								flex: 1;
								justify-content: flex-start;
							}
							.right {
								width: 50%;
							}
						}

						&-body {
							// padding-top: 30px;
							max-height: 300px;
							overflow: scroll;
						}
					}
				}
				// :deep (.operation) {
				//     text-align: center;
				// }
			}
		}
	}
</style>
