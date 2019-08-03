<template>
	<section>
		<el-row>
			<el-col class="left-container" :span="12">
				<el-tree :data="productTypes" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
			</el-col>
			<el-col class="right-container" :span="12">

			</el-col>
		</el-row>
	</section>
</template>



<script>
    export default {
        data() {
            return {
                productTypes: [],
                defaultProps: {
                    children: 'children',
                    label: 'name'
                }
            };
        },
        methods: {
            handleNodeClick(data) {
                console.log(data);
            },
			loadProductType(){
                //发送请求到后台，查询数据，将数据返回给前台页面展示
				this.$http.get("/product/productType/list")
					.then((res)=>{
					    let data = res.data;
					    this.productTypes = data;
					});
			}
        },
		//加载完毕之后，执行
		mounted(){
            this.loadProductType();
		}
    };
</script>

<style scoped>
	.left-container{
		/*background-color: #50bfff;*/
		height: 500px;
		border: 1px solid #d9d9d9;
	}
	.right-container{
		/*background-color: #42d885;*/
		height: 500px;
		border: 1px solid #d9d9d9;
		border-left: none;
	}

</style>