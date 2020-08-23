# 前言

考研加油，面试顺利~

## 修改日期

* 2020/8/22  v1.0


## 项目概述

该项目适用于高校，可为辅导员与学生之间的管理交流提高效率。其中包含辅导员管理端，学生端，兼容移动端，使用了Vue框架搭建，配合Element ui,vant组件库。 该项目最终成品包含：管理员PC端，学生PC端，学生移动端。

## 前端涉及技术

* Vue 2.0
* element ui组件
* vant ui组件
* ES6

## 相关文档

* <a href="https://cn.vuejs.org/" target="_blank">Vue.js</a>
* <a href="https://echarts.apache.org/examples/zh/index.html?theme=light" target="_blank">Echarts</a>
* <a href="https://element.eleme.cn/#/zh-CN" target="_blank">Element组件</a>
* <a href="https://youzan.github.io/vant/#/zh-CN/" target="_blank">Vant组件</a>

## 面试知识点

* <a href="https://www.jianshu.com/p/6ce352614928" target="_blank">vue项目搭建过程</a>
* <a href="https://www.jianshu.com/p/7dae4405baf5" target="_blank">Vue基础知识</a>
* <a href="https://www.jianshu.com/p/93fa30986d07" target="_blank">axios的用法与功能实现</a>
* <a href="https://www.jianshu.com/p/ac1787f6c50f" target="_blank">ES6中常用新特性</a>
* <a href="https://www.jianshu.com/p/2188dcd91090" target="_blank">浅拷贝与深拷贝</a>
* <a href="https://www.jianshu.com/p/2e69e9891c67" target="_blank">优化相关</a>
* <a href="https://www.cnblogs.com/yet-320/p/13540131.html" target="_blank">一次完整的HTTP请求流程</a>
* <a href="https://www.jianshu.com/p/b13d811a6a76" target="_blank">移动端适配方案</a>
* <a href="https://www.jb51.net/article/70673.htm" target="_blank">跨域相关</a>

# 辅导员管理端

## 项目结构

## 功能实现

### test

# 学生端

> 学生端包含移动端兼容，其原理为：登录页mounted初始周期通过JS对打开设备的型号判断进行不同的登录页渲染进入不同的路由页面。

## 项目结构
* public（项目入口文件）
* src（开发目录）
  * api（后端接口Api）
  * assets（静态样式，图片）
  * component（自定义封装组件）
    * admin（废弃）
    * common（布局组件）
    * module（展示组件）
  * config（代理配置项）
  * util（axios配置项）
  * view（视图页面）
    * admin（废弃）
    * home（移动端相关页面）
    * pchome（PC端相关页面）
    * profile（个人相关页面）
    * Login.vue（登录页）
    * Index.vue（废弃）
  * App.vue （Vue入口）
  * main.js （核心文件入口）
  * router.js （路由配置）
  * store.js （Vuex配置）
* test（测试规范）
* 其余（配置项文件）

## 功能实现

> 目前实现功能如下，可扩展学生课程表及成绩查询功能，动态通知发送功能等。

### 登录判断
> Login.vue文件登录代码
* 通过后端Api登录接口，将账号密码经过Post请求方法请求后端，后端进行判断处理后，将响应结果返回至前端，前端进行判断。由于经验不足，没有进行token加密的请求处理，由后端获取IP后存入临时表进行登录判断，15分钟后清除临时表可造成登录过期。

```javascript
    doLogin () {
      if (this.loginForm.account == '') {
        this.$toast('学号不能为空');
        return false
      }
      if (this.loginForm.password == '') {
        this.$toast('密码不能为空');
        return false
      }

      // 保存的账号
      var name = this.loginForm.account;
      // 保存的密码
      var pass = this.loginForm.password;
      var exdate = new Date();// 获取时间
      exdate.setTime(exdate.getTime() + 24 * 60 * 60 * 1000 * 360);// 保存的天数
      // 字符串拼接cookie
      window.document.cookie = 'account' + '=' + name + ';path=/;expires=' + exdate.toGMTString();
      if (this.checked === true) {
        // 传入账号名，密码，和保存天数3个参数
        this.setCookie(pass, 360);
      } else {
        this.clearCookie();
      }
      const result = loginApi.doLogin({
        account: this.loginForm.account,
        password: this.loginForm.password
      });

      result.then(res => {
        // console.log(res)
        if (res.status == 200 && res.info == 'Student') {
          localStorage.setItem('login_account', this.loginForm.account)
          localStorage.setItem('login_password', this.loginForm.password)
          if (this.pc === false && this.mobile === true) {
            this.$router.push('/student/home')
          } else if (this.pc === true && this.mobile === false) {
            this.$router.push('/student/pchome')
          } else { this.$router.push('/student/home') }
        } else if (res.status == 200 && res.info == 'Master') {
          alert('系统检测到该账号为管理员账号，请在登录按钮下方点击管理员登录入口进行登录！');
        } else { this.$router.push('/student/login') }
      }
      );
    }
```
### 查询个人信息

> profile/Index.vue移动端显示。Pcperinfo.vue PC端显示。 
* 通过获取个人信息接口可查看个人信息。
```javascript
getUserInfo: function () {
      const get = getStu.getStuInfo();
      get.then(res => {
        if (res.data !== undefined) {
          this.info.id = res.data.stuId
          this.info.name = res.data.stuName
          this.info.academic = res.data.stuAcademic
          this.info.major = res.data.stuMajor
          this.info.class = res.data.stuClass
          this.info.dorm = res.data.stuDorm
          // this.info.score = res.data.stuScore
          if (res.data.monitor == 1) { this.info.monitor = '任职中' } else { this.info.monitor = '否' }
          if (res.data.dormHeader == 1) { this.info.dormHeader = '任职中' } else { this.info.dormHeader = '否' }
          //  if(res.data.free == 1){ this.info.free = '已转出'}
          // else {this.info.free = '未转' }
        }
      })
    }
```
### 查询文明分情况
> ScoreTable.vue PC端文明分表格组件封装。home/Index.vue 移动端文明分查询页面
* PC端通过element的table表格组件进行文明分的记录展示。
```html
<el-table
    v-loading="loading"
    :data="tableData.filter(data =>
            (!history && data.currentSemester === currentSemester)
            || (history && data.currentSemester !== currentSemester))"
    style="min-width:950px;width: 100%;border:2px solid #f8f8f8;"
  >
    <el-table-column
      prop="detailName"
      label="项目"
    >
      <template slot-scope="scope">
        <el-tag size="medium">{{ scope.row.detailCategory }}</el-tag>&nbsp;
        <span>{{ scope.row.detailName }}</span>
      </template>
    </el-table-column>
    <el-table-column
      prop="score"
      label="分值"
      sortable
      width="80px"
    >
      <template slot-scope="scope">
        <span :style="{'color':(scope.row.score >= 0.0 ? '#47DB47' :'red')}"><strong>{{ scope.row.score }}</strong></span>
      </template>
    </el-table-column>
    <el-table-column
      prop="remarks"
      label="备注"
    >
      <template slot-scope="scope">
        <span v-if="scope.row.remarks !== ''">{{ scope.row.remarks }}</span>
        <el-tag size="mini" v-else type="info">无备注</el-tag>
      </template>
    </el-table-column>
    <el-table-column
      prop="createTime"
      label="日期"
      sortable
      width="200"
    >
      <template slot-scope="scope">
        <span style="margin-left: 10px">{{ formatTime(scope.row.createTime) }}</span>
      </template>
    </el-table-column>
    <el-table-column label="学期" width="180" v-if="history">
      <template slot-scope="scope">
        <span>{{scope.row.currentSemester}}</span>
      </template>
    </el-table-column>
    <el-table-column label="记录人" width="150">
      <template slot-scope="scope">
        <span>{{ scope.row.masName }}</span>&nbsp;
        <el-tag size="mini" type="info">{{ scope.row.masAccount }}</el-tag>
      </template>
    </el-table-column>
    <el-table-column
      label="操作"
      width="120"
    >
      <template slot-scope="scope">
        <Appeal
          :team-code="scope.row.teamCode"
          v-if="!history && scope.row.appeal === 'none' && scope.row.cancel === '0'"
        ></Appeal>
        <el-tag size="mini" v-else-if="scope.row.appeal === 'underway'">申诉中</el-tag>
        <el-tag size="mini" v-else-if="scope.row.appeal === 'success'" type="success">申诉成功</el-tag>
        <el-tag size="mini" v-else-if="scope.row.appeal === 'failure'" type="danger">申诉失败</el-tag>
        <el-tag size="mini" v-else-if="scope.row.appeal === 'termination'" type="success">申诉过程中撤回</el-tag>
        <el-tag size="mini" v-else-if="scope.row.cancel === '1'" type="warning">已撤回</el-tag>
        <el-tag size="mini" v-else type="info">历史记录无法申诉</el-tag>
      </template>
    </el-table-column>
  </el-table>
```
* 移动端通过自定义盒子利用v-for循环进行记录渲染。
```html
<div class="home-tab">
        <van-tabs
          type="line"
          color="#4169e1"
          border="flase"
        >
          <van-tab title="全部记录">
            <ul>
              <li v-for="(k,item) in  scoreData" :key="item">
                <hr class="line">
                <ScoreCard
                  :detailCategory="k.detailCategory"
                  :detailName="k.detailName"
                  :time="timestampFormat(new Date(k.createTime.replace(/-/g, '/')).getTime()/1000)"
                  :message="k.remarks"
                  :score="k.score"
                  @click.native="doshow(k.detailName,k.remarks,k.createTime)"
                >
                </ScoreCard>
              </li>
            </ul>
          </van-tab>
          <van-tab title="加分记录">
            <ul>
              <li v-for="(i,item) in  scoreAdd" :key="item">
                <hr class="line">
                <ScoreCard
                  :detailCategory="i.detailCategory"
                  :detailName="i.detailName"
                  :time="timestampFormat(new Date(i.createTime.replace(/-/g, '/')).getTime()/1000)"
                  :message="i.remarks"
                  :score="i.score"
                  @click.native="doshow(i.detailName,i.remarks,i.createTime)"
                >
                </ScoreCard>
              </li>
            </ul>
          </van-tab>
          <van-tab title="减分记录">
            <ul>
              <li v-for="(j,item) in  scoreSub" :key="item">
                <hr class="line">
                <ScoreCard
                  :detailCategory="j.detailCategory"
                  :detailName="j.detailName"
                  :time="timestampFormat(new Date(j.createTime.replace(/-/g, '/')).getTime()/1000)"
                  :message="j.remarks"
                  :score="j.score"
                  @click.native="doshow(j.detailName,j.remarks,j.createTime)"
                >
                </ScoreCard>
              </li>
            </ul>
          </van-tab>
        </van-tabs>
        <p class="home-bottom">已全部加载~</p>
      </div>
```

### 文明分申诉

> component/module/Appeal.vue
* 可在表格操作内对当前文明分记录进行申述。
```javascript
 submit: function () {
        const url = '/appealScore/add'
        const data = `teamCode=${this.teamCode}&state=${this.state}`
        $http.post(url, data).then(res => {
          this.$message({
            message: res.info,
            type: 'success',
            duration: 5000,
            position: 'top-left',
          })
        })
        this.appealDialogVisible = false
      },
```
### 留言反馈
> pchome/qcxbook.vue PC端显示。phoneqcxbook.vue 移动端显示。
* 该模块由于后端开发人员不足，故由前端开发人员进行开发，由于不涉及后台Api,就采用了传统的form表单提交的action方式进行信息的提交，提交的地址为PHP文件，PHP文件经过处理后写入数据库，完成留言反馈的提交。
```html
<form
          action="http://120.78.124.142/student/pchome/qcxadd.php"
          method="post"
          id="commentform"
        >
```

### 团队报名
> profile/join.vue移动端显示。Pcjoin.vue PC端显示。 
* 同留言反馈。

### PHP文件源码
```php
```