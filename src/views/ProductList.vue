<template>
    <div>
      <b-container fluid>
			<b-row>
				<b-col style="display:flex;justify-content: space-between;margin: 15px 0;">
            <h2>产品列表</h2>
            <div>
              <b-button @click="refreshHandle">刷新</b-button>
							<b-button variant="info" v-b-modal="'new_product_modal'">添加产品</b-button>
            </div>
        </b-col>
			</b-row>
			<b-row>
				<b-col md="12">
					<b-table stacked="sm" striped hover :items="product_table_items" :fields="product_table_fields">
						<template slot="action" slot-scope="row">
							<b-button variant="info" @click="checkHandle(row.item, row.index, $event.target)">查看</b-button>
							<b-button @click="deviceHandleDelete(row.item)">删除</b-button>
						</template>
						<template slot="add_time"  slot-scope="row">
							{{moment(row.item.add_time).format('llll')}}
						</template>
					</b-table>
				</b-col>
			</b-row>
		</b-container>
    <b-modal 
			id="new_product_modal" 
			ref="new_product_modal"
			title="新建产品" 
			cancel-title="取消" 
			ok-title="确定" 
			ok-variant="info"
			@show="ProductShow"
			@ok="newProductHandleOk"
			>
      <b-container fluid>
        <b-row>
          <b-col>
            <b-form ref="product_info_form" @submit.stop.prevent="handleSubmit">
              <b-form-group :state="productTitleStateValidation" label-for="product-title-input" invalid-feedback="产品名称不能为空">
								<div slot="label">
									<small style="color:red;">*</small>
									产品名称
								</div>
                <b-form-input 
									id="product-title-input" 
									v-model="productTitle" 
									:state="productTitleStateValidation" 
									required/>
              </b-form-group>
              <b-form-group :state="productTypeStateValidation" label-for="product-type-select" invalid-feedback="请选择所属分类">
								<div slot="label">
									<small style="color:red;">*</small>
									产品分类
								</div>
                <b-form-select id="product-type-select" v-model="productType" :state="productTypeStateValidation" :options="product_type_items"></b-form-select>
              </b-form-group>
            </b-form>
          </b-col>
        </b-row>
      </b-container>
      <p>节点类型：</p>
      <b-container fluid>
        <b-row>
          <b-col>
            <b-form ref="form" @submit.stop.prevent="handleSubmit">
              <b-form-group label-for="name-input">
								<div slot="label">
									<small style="color:red;">*</small>
									节点类型
								</div>
                <b-form-radio-group v-model="nodeType" :options="node_type" plain>
								</b-form-radio-group>
              </b-form-group>
            </b-form>
          </b-col>
        </b-row>
      </b-container>
      <p>联网与数据：</p>
			<b-container fluid>
        <b-row>
          <b-col>
						<b-form ref="form" @submit.stop.prevent="handleSubmit">
							<b-form-group :state="internetDelectStateValidation" label-for="name-input">
								<div slot="label">
									<small style="color:red;">*</small>
									联网方式
								</div>
								<b-form-select :state="internetDelectStateValidation" v-model="internetDelect" :options="internet_delect"></b-form-select>
							</b-form-group>
						</b-form>
          </b-col>
        </b-row>
			</b-container>
    </b-modal>
    </div>
</template>
<script>
import randomString from 'random-string';
import axios from 'axios'
import { get, post, axiosdelete } from '../request'
export default {
  data() {
    return {
			productTitle: '',     // 产品名称
			productType: null,    // 产品类型
			nodeType: '设备',   // 节点类别
			internetDelect: 'wifi',  // 接入网络
      product_type_items: [
        { text: "智能城市", value: "city" },
        { text: "智能生活", value: "life" },
        { text: "智能农业", value: "agriculture" },
        { text: "智能工业", value: "industry" }
      ],
      node_type: [
        { text: "网关", value: "网关" },
        { text: "设备", value: "设备" }
      ],
      internet_delect: [
        { value: "wifi", text: "WiFi" },
        { value: "data_traffic", text: "蜂窝（2G/3G/4G）" },
        { value: "ethernet", text: "以太网" }
      ],
      product_table_fields: [
        {
          key: "product_title",
          label: "产品名称",
          sortable: false
        },
        {
          key: "productkey",
          label: "ProductKey",
          sortable: false
        },
        {
          key: "node_type",
          label: "节点类型",
          sortable: false
        },
        {
          key: "add_time",
          label: "添加时间",
          sortable: false
        },
        {
          key: "action",
          label: "操作"
        }
      ],
      product_table_items: [
        // {
        //   product_title: "测试",
        //   productkey: "xxxxxxxx",
        //   node_type: "设备",
        //   add_time: "2019/05/27 20:48:30"
        // }
      ]
    }
  },
  computed: {
		productTitleStateValidation() {
			return this.productTitle.length >= 4 && this.productTitle.length < 30
		},
		productTypeStateValidation() {
			return this.productType != null
		},
		internetDelectStateValidation() {
			return this.internetDelect != null
		}
  },
  methods: {
		// checkFormValidity() {
		// 	const valid = this.$refs.product_info_form.checkValidity()
		// 	console.log(valid)
		// 	this.productTitleState = valid ? 'valid' : 'invalid'
		// 	return valid
		// },
		ProductShow() {
			this.productTitle = '';
			this.productType = null;
			this.nodeType = 'device';
			this.internetDelect = 'wifi';
		},
		newProductHandleOk(bvModalEvt) {
			bvModalEvt.preventDefault()
			this.handleSubmit()
		},
		handleSubmit() {
			// Exit when the form isn't valid
			if (!this.productTitleStateValidation || !this.productTypeStateValidation) {
				return
			}

			const productkey = randomString({length: 11})

			// console.log({
			// 	product_title: this.productTitle,
			// 	productkey: productkey,
			// 	node_type: this.nodeType,
			// 	add_time: this.moment(new Date()).format('llll')
			// })

			post('/product',{
				product_title: this.productTitle,
				productkey: productkey,
				node_type: this.nodeType,
				add_time: this.moment(new Date())
			})
			.then( data => {
				console.log(data)
				this.product_table_items = data;
			})
			.catch( err => {
				console.log(err)
			})

			// Push the name to submitted names
			// this.submittedNames.push(this.name)
			// Hide the modal manually
			this.$nextTick(() => {
				this.$refs.new_product_modal.hide()
			})
		},
		refreshHandle() {
			get('/product')
			.then( data => {
				console.log(data)
				this.product_table_items = data;
			})
			.catch( err => {
				console.log(err)
			})
		},
		checkHandle(item, index, target) {
			console.log(item, index, target)
			this.$router.push({
				path: `product/detail/${item.productkey}`,
			})
		},
		deviceHandleDelete(item) {
      console.log(item)
      axiosdelete(`/product/${item.productkey}`)
      .then( res => {
        console.log(res);
        this.refreshHandle();
      })
      .catch( err => {
        console.error(err);
      })
    },
  },
  created(){

		console.log(JSON.parse(localStorage.getItem('user')).token)


		get('/product')
		.then(data => {
			console.log(data)
			this.product_table_items = data;
		})
		.catch(err => console.log);
	},
};
</script>
<style lang="scss" scoped>

</style>

