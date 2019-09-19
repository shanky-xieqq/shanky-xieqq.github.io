---
layout: post
title:  "element-ui表单渲染v-if组件验证时报错"
categories: Linux
tags: vue element-ui
---

* content
{:toc}

使用elementui的form表单组件，如果其中几个表单项使用了v-if进行UI切换，并且默认`v-if="false"`不可见，切换的元素又是必填项时，那么在验证的时候，就会出现很奇怪的bug。				          
		   					    
				




### 原因

* elementui在对form表单中带有prop属性的子组件进行校验规则绑定时，是在vue声明周期mounted完成的。而v-if用来切换的元素是会被销毁的，导致了v-if内的表单项，由于在mounted时期没有进行渲染，所以规则也没有绑定上。

* 那么就对el-form-item使用rules来进行动态绑定，一旦切换就会绑定上这个rules。


* 另一个坑，自定义的验证方法中，比如`validLandlordName(rule, value, callback) {}`，有时会出现value这个值取不到为空。那么就直接获取你输入的属性值`this.houseInfo.landlord.landlordName`来进行验证，不要再对value进行判断了。

```html
<el-form ref="form" :rules="rules" :model="houseInfo" inline size="mini" label-width="150px">
      <el-row>
        <el-col>
          <el-form-item label="所属社区" label-width="150px" prop="communityId">
            <el-select v-model.trim="houseInfo.communityId" placeholder="所属社区" :disabled="true">
              <el-option v-for="item in communityList" :value="item.communityId" :label="item.communityName" :key="item.communityId">
              </el-option>
            </el-select>
          </el-form-item>
        </el-col>
      </el-row>
    <!--  houseInfo.houseType 值为4-->
    <el-row v-if="houseInfo.houseStatus == 4">
        <el-col>
          <el-form-item label="所属楼栋/单元" prop="buildingId" label-width="150px">
            <el-cascader placeholder="所属楼栋/单元" :disabled="houseTypeDisabled" :options="deptTreeBuilding" v-model="houseInfo.selectOptions"
              @change="selectedBuildingChanged" :change-on-select="true" style="width:200px">
            </el-cascader>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
      <!-- 此时这个手机号码，默认是不显示的，一旦切换渲染出来，验证规则不做特殊处理就会报错-->
      <div v-if="houseInfo.houseStatus !== 4">
        <el-row>
          <el-col>
            <el-form-item label="手机号码" prop="landlord.contactMobile" >
              <el-input v-model="houseInfo.landlord.contactMobile" style="width:200px" placeholder="请输入手机号码"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
      </div>
</el-form>
```

### 解决办法

* v-if中的表单元素，使用动态验证规则绑定，：rule=[XXX]，同时在method中自定义验证方法即可。

![](/img/img20190826_1.png)    

![](/img/img20190826_2.png)    


   













