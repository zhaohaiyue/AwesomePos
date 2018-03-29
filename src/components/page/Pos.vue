<template>
  <div class="pos">
      <el-row>
        <el-col :span="7" class='post-order' id='order-list'>
          <el-tabs type='card'>
            <el-tab-pane label='点餐'>
              <el-table :data='tableData' border style="width: 100%">
                <el-table-column prop='goodsName' label='商品名称'  align='center'></el-table-column>
                <el-table-column prop='count' label='数量' align='center'></el-table-column>
                <el-table-column prop='price' label='金额' align='center'></el-table-column>
                <el-table-column  label='操作'  align='center' fixed='right'>
                  <template scope='scope'>
                    <el-button type='text' size='small' @click='delSingleGoods(scope.row)'>删除</el-button>
                    <el-button type='text' size='small' @click='addOrderList(scope.row)'>添加</el-button>
                  </template>
                </el-table-column>
              </el-table>
              <div class='totalDiv'> 
                <small>数量：</small>{{totalCount}}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                <small>金额: </small>{{totalMoney}}
              </div>
              <div class='btn'>
                <el-button type='warning'>挂单</el-button>
                <el-button type='danger' @click="delAllGoods">删除</el-button>
                <el-button type='success' @click="checkout">结账</el-button>
              </div>
            </el-tab-pane>
            <el-tab-pane label='挂单'>
              
            </el-tab-pane>
            <el-tab-pane label='外卖'>
              
            </el-tab-pane>
          </el-tabs>
        </el-col>
        <el-col :span="17">
          <div class="often-goods">
            <div class="title">常用商品</div>
            <div class='often-goods-list'>
              <ul>
                <li v-for="item in oftenGoods" @click="addOrderList(item)">
                  <span>{{item.goodsName}}</span>
                  <span class="o-price">&yen;{{item.price}}元</span>
                </li>
              </ul>
            </div>
          </div>
          <div class='goods-type'>
            <el-tabs type="card">
              <el-tab-pane label='汉堡'>
                <div>
                  <ul class="cookList">
                     <li v-for="item in type0Goods" @click="addOrderList(item)">
                          <span class="foodImg"><img :src="item.goodsImg" width="100%"></span>
                          <span class="foodName">{{item.goodsName}}</span>
                          <span class="foodPrice">&yen;{{item.price}}元</span>
                      </li>
                  </ul>
                </div>
              </el-tab-pane>
              <el-tab-pane label='小食'>
                <ul class="cookList">
                   <li v-for="item in type1Goods" @click="addOrderList(item)">
                      <span class="foodImg"><img :src="item.goodsImg" width="100%"></span>
                      <span class="foodName">{{item.goodsName}}</span>
                      <span class="foodPrice">&yen;{{item.price}}元</span>
                    </li>
                </ul>
              </el-tab-pane>
              <el-tab-pane label='饮料'>
                <ul class="cookList">
                  <li v-for="item in type2Goods" @click="addOrderList(item)">
                      <span class="foodImg"><img :src="item.goodsImg" width="100%"></span>
                      <span class="foodName">{{item.goodsName}}</span>
                      <span class="foodPrice">&yen;{{item.price}}元</span>
                  </li>
                </ul>
              </el-tab-pane>
              <el-tab-pane label='套餐'>
                <ul class="cookList">
                  <li v-for="item in type3Goods" @click="addOrderList(item)">
                      <span class="foodImg"><img :src="item.goodsImg" width="100%"></span>
                      <span class="foodName">{{item.goodsName}}</span>
                      <span class="foodPrice">&yen;{{item.price}}元</span>
                  </li>
                </ul>
              </el-tab-pane>
            </el-tabs>
          </div>
        </el-col>
      </el-row>
  </div>
</template>
<script>
import axios from "axios";
export default {
  name: "pos",
  data() {
    return {
      tableData: [],
      oftenGoods: [],
      type0Goods: [],
      type1Goods: [],
      type2Goods: [],
      type3Goods: [],
      totalMoney: 0,
      totalCount: 0
    };
  },
  created: function() {
    axios
      .get("http://jspang.com/DemoApi/oftenGoods.php")
      .then(res => {
        if (res.status === 200) {
          console.log(res);
          this.oftenGoods = res.data;
        }
      })
      .catch(err => {
        alert("网络错误,不能访问");
      });

    axios.get("http://jspang.com/DemoApi/typeGoods.php").then(res => {
      console.log(res);
      this.type0Goods = res.data[0];
      this.type1Goods = res.data[1];
      this.type2Goods = res.data[2];
      this.type3Goods = res.data[3];
    });
  },
  mounted: function() {
    let orderHeight = document.body.clientHeight;
    document.getElementById("order-list").style.height = orderHeight + "px";
  },
  methods: {
    //添加订单列表的方法
    addOrderList(goods) {
      this.totalMoney = 0;
      this.totalCount = 0;

      let isHave = false;
      //判断这个商品是否已经存在订单列表中
      for (let i = 0; i < this.tableData.length; i++) {
        if (this.tableData[i].goodsId == goods.goodsId) {
          isHave = true; //存在
        }
      }
      if (isHave) {
        //存在就进行数量添加
        let arr = this.tableData.filter(o => o.goodsId == goods.goodsId);
        arr[0].count++;
      } else {
        //不存在就推入数组中
        let newGoods = {
          goodsId: goods.goodsId,
          goodsName: goods.goodsName,
          price: goods.price,
          count: 1
        };
        this.tableData.push(newGoods);
      }
      this.getAllMoney();
    },
    delSingleGoods(goods) {
      this.tableData = this.tableData.filter(o => o.goodsId != goods.goodsId);
      this.getAllMoney();
    },
    getAllMoney() {
      this.totalMoney = 0;
      this.totalCount = 0;
      if (this.tableData) {
        //计算金额和数量
        this.tableData.forEach(ele => {
          this.totalCount += ele.count;
          this.totalMoney = this.totalMoney + ele.price * ele.count;
        });
      }
    },
    delAllGoods() {
      this.tableData = [];
      this.totalMoney = 0;
      this.totalCount = 0;
    },
    checkout() {
      if(this.totalCount != 0){
        this.tableData = [];
        this.totalMoney = 0;
        this.totalCount = 0;
        this.$message({
          message:'结账成功',
          type:'success'
        })
      }else{
        this.$message.error('不能空结')
      }
    }
  }
};
</script>

<style scoped>
.post-order {
  background-color: #f9fafc;
  border-right: 1px solid #c0ccda;
  height: 100%;
}
.btn {
  display: table;
  margin: 10px auto;
}
.title {
  height: 20px;
  border-bottom: 1px solid #d3dce6;
  background-color: #f9fafc;
  padding: 10px;
}
.often-goods-list ul li {
  list-style: none;
  float: left;
  border: 1px solid #d3dce6;
  padding: 10px;
  margin: 10px;
  background-color: #fff;
  cursor: pointer;
}
.o-price {
  color: #58b7ff;
}
.goods-type {
  clear: both;
  padding-top: 20px;
}
.cookList li {
  list-style: none;
  width: 23%;
  border: 1px solid #e5e9f2;
  height: auto;
  overflow: hidden;
  background-color: #fff;
  padding: 2px;
  float: left;
  margin: 2px;
  cursor: pointer;
}
.cookList li span {
  display: block;
  float: left;
}
.foodImg {
  width: 40%;
}
.foodName {
  font-size: 16px;
  padding-left: 10px;
}
.foodPrice {
  font-size: 16px;
  color: brown;
  padding-left: 10px;
  padding-top: 10px;
}
.totalDiv {
  text-align: center;
  border-bottom: 1px solid #e5e9f2;
  padding: 10px;
}
</style>


