
<style>

    #uploadBox
    {
        height: 260px;
        background-color: #fff;
        border: 3px dashed rgb(119, 119, 119);
        line-height: 250px;
        /* margin: 120px auto; */
        text-align: center;
        /* line-height: 362px; */
        font-size: 1.5em;
        font-weight: bold;
        color: #444;
        overflow-y: auto;
    }

    #uploadBox ul li
    {
        float: left;
        position: relative;
        margin-left: 5px;
        margin-top: 5px;
    }

    #uploadBox  li img
    {
        border: 1px solid #D1D1D1;
        width: 198px;
        height: 112px;
        vertical-align: middle;
    }

    #uploadBox  li  .percentage
    {
        width: 69px;
        height: 69px;
        position: absolute;
        left: 50%;
        top: 50%;
        margin-left: -34.5px;
        margin-top: -34.5px;
        text-align: center;
        font-size: 18px;
        line-height: 69px;
        color: #fff;
        border-radius: 34.5px;
        background: rgba(0, 0, 0, .8);
    }

    #uploadBox  li.done .percentage
    {
        background: url("../assets/images/done.png") no-repeat 0 0;
    }

    #uploadBox li .progress
    {
        position: absolute;
        bottom: 0px;
        width: 200px;
        background: #000;
        opacity: .5;
        left: 0;
    }

    #uploadBox  li.done .progress
    {
        display: none;
    }

    .clearfix
    {
        clear: both;
    }

    .hidden
    {
        display: none;
    }
    .clear{
        position: relative;
    }

    .clear:after {content: "";display: table;clear: both;}

    #image_manager .img_delete_btn_box{
        border-bottom:1px solid #e3e8ee;
        padding: 14px 0px;
    }


    #image_manager .img-content{
        margin: 0px;
    }
    #image_manager .image-item>li>.img-content>.imgbox>img {
        width: 100%;
    }

    #image_manager .image-item>li>.img-content>.delete {
        width: 28px;
        position: absolute;
        z-index: 999;
        bottom: 0px;
        right: 0px;
    }

    .image-item li {
        /* overflow: hidden; */
        padding: 2px;
        height: 216px;
        border: 2px solid #eee;
        margin: 0px 10px 25px 10px;
        background: #fff;
        float: left;
        width: -moz-calc(100% - 80px);
        width: -webkit-calc(100% - 80px);
        width: calc(25% - 28px);
    }

    #image_manager .image-item li.selected {
        border-color: #f54015;
    }

    #image_manager .image-item li.selected .delete {
        display: block;
        cursor: pointer;
    }
    #image_manager .image-item li div{
        width:100%;
        overflow:hidden;
        position: relative;
    }

    #image_manager .image-item li .imgbox div {
        position: absolute;
        top: 0;
        height: 100%;
        background: rgba(0,0,0,0.3);
        padding-top: 44px;
        color: #fff;
        overflow: auto;
        display: none;
        font-size:32px;
    }

    #image_manager .image-item li .imgbox:hover div {
        display: block;
    }

    #image_manager .img_list_box {

    }
    #image_manager .image-item {
        padding: 25px 0;
        width: -moz-calc(100% - 30px);
        width: -webkit-calc(100% - 30px);
        width: calc(100% - 30px);
        margin: auto;
        text-align:center;
    }


    #image_manager .img-content{
        color: #666;
        height: 209px;
        overflow: visible !important;
    }

    #image_manager .image-item .imgbox {
        height: 134px;
        padding-bottom: 3px;
        position: relative;
    }

    #image_manager .img-title {
        border-top: 1px solid #eee;
        font-size: 16px;
        line-height: 30px;
        color:#999;
    }
    #image_manager .img-time {
        color:#ccc;
        white-space:nowrap;
        font-size:12px;
    }

    #image_manager .img-link {
        cursor: pointer;
        color: #9c9cff;
    }
    #image_manager .img-linkbox input{
        width: 100%;
    }

    .image_manager_show_big_img{
        width:calc(90% - 300px);
        left:calc(5% + 300px);
        top:100px;
        position: fixed;
        z-index: 99;
    }
</style>
<script>
  import injection from '../helpers/injection';
  import $ from '../../node_modules/jquery';

  export default {
      beforeRouteEnter(to, from, next) {
          injection.loading.start();
          injection.http.get(`${window.api}/images/lists`).then(response => {
              const imglist = response.data.data;
              const pagination = response.data.pagination;
              next(vm => {
                  imglist.forEach(image => {
                      image.deleteSelector = false;
                      image.showLinkStatus = false;
                  });
                  vm.imglist = imglist;
                  vm.pagination = pagination;
                  injection.message.info('获取图片列表成功！');
                  injection.sidebar.active('setting');
              });
          }).catch(() => {
              injection.loading.fail();
          });
      },
      data() {
          return {
              imglist: [],
              pagination: { current_page: 1 },
              deleteStatus: false,
              DelBtnText: '删除图片',
              id: 1,
              BigImagesPath: '',
              showBigimgStatus: false,
              UploadTips: '将文件拖拽至此区域，即可上传！',
              percent: 0,
          };
      },
      methods: {
          deleteSelector(index) {
              const self = this;
              if (self.deleteStatus) {
                  const image = self.imglist[index];
                  image.deleteSelector = !image.deleteSelector;
                  console.log(image);
              }
              console.log(self);
          },
          deleteimg() {
              const self = this;
              self.deleteStatus = !self.deleteStatus;
              if (!self.deleteStatus) {
                  self.imglist.forEach(image => {
                      image.deleteSelector = false;
                  });
                  self.DelBtnText = '删除图片';
              } else {
                  self.DelBtnText = '放弃删除';
              }
          },
          showLink(index) {
              const self = this;
              const images = self.imglist[index];
              images.showLinkStatus = true;
          },
          hiddenLink(index) {
              const self = this;
              const images = self.imglist[index];
              images.showLinkStatus = false;
          },
          paginator(id) {
              const self = this;
              self.id = id;
              self.$loading.start();
              self.$http.get(`${window.api}/images/lists?page=${id}`).then(response => {
                  const imglist = response.data.data;
                  imglist.forEach(image => {
                      image.deleteSelector = false;
                      image.showLinkStatus = false;
                  });

                  self.imglist = imglist;
                  self.pagination = response.data.pagination;
                  self.$loading.finish();
                  self.$message.info('更新图片列表成功！');
              }).catch(() => {
                  self.$loading.fail();
              });
          },
          deleteImages() {
              const self = this;
              if (!self.deleteStatus) {
                  self.$message.info('请在删除状态下提交！');
              } else {
                  const deleteIds = [];
                  self.imglist.forEach(image => {
                      if (image.deleteSelector) {
                          deleteIds.push(image.id);
                      }
                  });
                  self.$loading.start();
                  self.$http.delete(`${window.api}/imagemanager/delete?ids=${deleteIds}`).then(response => {
                      self.$loading.finish();
                      self.$message.info(response.data.message[0]);
                      self.paginator(self.id);
                  }).catch(() => {
                      self.$loading.fail();
                  });
              }
          },
          showBigimg(index) {
              const self = this;
              if (!self.deleteStatus) {
                  self.showBigimgStatus = true;
                  self.BigImagesPath = self.imglist[index].path;
              }
          },
          hideBigimg() {
              const self = this;
              self.showBigimgStatus = false;
          },
          uploadOndragenter() {
              const self = this;
              self.UploadTips = '释放鼠标立即上传!';
          },
          uploadOndragover(ev) {
              ev.preventDefault();
              self.UploadTips = '将文件拖拽至此区域，即可上传!';
          },
          uploadOndragleave() {
              const self = this;
              self.UploadTips = '将文件拖拽至此区域，即可上传!';
          },
          uploadOndrop(ev) {
              const self = this;
              self.UploadTips = '将文件拖拽至此区域，即可上传!';
              ev.preventDefault();
              const files = ev.dataTransfer.files;
              $.each(files, (k, file) => {
                  self.showUploadFile(file, self);
              });
          },
          showUploadFile(file, self) {
              const reader = new FileReader();
//            console.log(file);
//        console.log(reader);

            // 判断文件类型
              if (file.type.match(/image*/)) {
                  reader.onload = (e => {
                      const formData = new FormData();
                      const DomUploadEle = '#uploadBox';
                      const li = $('#template li').clone();
                      const img = li.find('img');
//                      const progress = li.find('.progress');
                      const percentage = li.find('.percentage');
                      percentage.text('0%');
                      img.attr('src', e.target.result);
                      $('ul', $(DomUploadEle)).append(li);
                      formData.append('uploadFile', file);
                // 上传文件到服务器
                      self.uploadToServer(formData);
                  });
                  reader.readAsDataURL(file);
              } else {
                  console.log(`此${file.name}不是图片文件！`);
              }
          },
          uploadToServer(formData) {
              const self = this;
              self.UploadTips = '正在上传图片!';
              self.$loading.start();
              self.$http.post(`${window.api}/imagemanager/upload`, formData).then(response => {
                  self.$loading.finish();
                  self.$message.info(response.data.message[0]);
                  self.UploadTips = '释放鼠标立即上传!';
                  self.paginator(self.id);
              }).catch(() => {
                  self.UploadTips = '将文件拖拽至此区域，即可上传!';
                  self.$loading.fail();
              });
          },
      },
  };


</script>


<template>



    <card>
        <img class="image_manager_show_big_img" v-if="showBigimgStatus" v-bind:src="BigImagesPath" v-on:click="hideBigimg()" />
        <div id="image_manager">
            <tabs value="images-list">
                <tab-pane label="图片列表" name="images-list">
                    <div class="img_delete_btn_box">
                        <i-button v-bind:class="{'ivu-btn-primary':deleteStatus, 'ivu-btn-dashed':!deleteStatus}" v-on:click="deleteimg()" >{{DelBtnText}}</i-button>
                        <i-button type="success" v-on:click="deleteImages()" v-bind:class="{'hidden':!deleteStatus}">确认删除</i-button>
                    </div>
                    <div class="img_list_box">
                        <ul class="image-item clear">
                            <li v-for="(imgobj, key) in imglist"  v-bind:class="{ 'selected': imgobj.deleteSelector }" v-on:click="deleteSelector(key)" >
                                <div class="img-content">
                                    <img v-if="!imgobj.deleteSelector && deleteStatus" class="delete" src="../assets/images/delete_corner_gray.png">
                                    <img v-if="imgobj.deleteSelector && deleteStatus" class="delete" src="../assets/images/delete_corner_red.png">
                                    <div v-on:click="showBigimg(key)" class="imgbox">

                                        <img  v-bind:src="imgobj.path" />
                                            <div>ID:{{imgobj.id}}</div>
                                    </div>
                                    <div class="img-title"> {{imgobj.title?imgobj.title:'无标题'}}</div>
                                    <div class="img-time"> {{imgobj.created_at}}</div>
                                    <div class="img-linkbox">
                                        <div v-bind:class="{'hidden':imgobj.showLinkStatus}" class="img-link" v-on:click="showLink(key)" >查看链接</div>
                                        <input v-bind:class="{'hidden':!imgobj.showLinkStatus}"  v-bind:value="imgobj.path" v-on:blur="hiddenLink(key)" />
                                    </div>
                                </div>
                            </li>
                        </ul>

                    </div>
                     <div class="ivu-page-wrap" v-bind:class="{'hidden':deleteStatus}">
                        <page :current="pagination.current_page" :page-size="pagination.per_page" :total="pagination.total" @on-change="paginator"></page>
                    </div>
                </tab-pane>
                <tab-pane label="上传图片" name="images-upload">

                    <div id="uploadBox" ref="uploadBox" v-on:drop="uploadOndrop" v-on:dragleave="uploadOndragleave" v-on:dragover="uploadOndragover" v-on:dragenter="uploadOndragenter">
                        {{UploadTips}}
                    </div>

                    <div id="template" class="hidden">
                        <li>
                            <img src=""/>
                            <span class="progress"></span>
                            <span class="percentage"></span>
                        </li>
                    </div>
                </tab-pane>
            </tabs>
        </div>
    </card>
</template>
