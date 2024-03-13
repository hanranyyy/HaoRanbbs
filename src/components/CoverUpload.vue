<!-- 封面上传 -->
<!-- 在EditPost中使用 -->
<template>
    <div class="cover-upload">
      <!-- :show-file-list="false"  上传文件后不会显示文件列表 -->
      <!-- :multiple="false"  禁止选择多个文件 -->
      <!-- :http-request="uploadImage"  上传文件时调用uploadImage -->
      <el-upload
        name="file"
        :show-file-list="false"
        accept=".png,.PNG,.jpg,.JPG,.jpeg,.JPEG,.gif,.GIF,.bmp,.BMP"
        :multiple="false"
        :http-request="uploadImage"
      >
        <div class="cover-upload-btn"> 
          <!-- 如果是本地图片 -->
          <template v-if="localFile">
            <img :src="localFile" />
          </template>
          <template v-else>
            <img
              :src="
                (imageUrlPrefix ? imageUrlPrefix : proxy.globalInfo.imageUrl) +
                modelValue.imageUrl
              "
              v-if="modelValue && modelValue.imageUrl"
            />
            <!-- 没有值显示空的图片 -->
            <span class="iconfont icon-add" v-else></span>
          </template>
        </div>
      </el-upload>
    </div>
  </template>
  
  <script setup>
  import { ref, reactive, getCurrentInstance } from "vue";
  import { useRouter, useRoute } from "vue-router";
  const { proxy } = getCurrentInstance();
  const router = useRouter();
  const route = useRoute();
  
  const props = defineProps({
    // 图片的地址前缀
    imageUrlPrefix: {
      type: String,
    },
    // 组件所需的数据对象
    modelValue: {
      type: Object,
      default: null,
    },
  });
  
  const localFile = ref(null);
  const emit = defineEmits();
  const uploadImage = async (file) => {
    file = file.file;
    let img = new FileReader();
    img.readAsDataURL(file);
    img.onload = ({ target }) => {
      localFile.value = target.result;
    };
    emit("update:modelValue", file);
  };
  </script>
  
  <style lang="scss">
  .cover-upload {
    .cover-upload-btn {
      background: rgb(245, 245, 245);
      width: 150px;
      height: 150px;
      display: flex;
      align-items: center;
      justify-content: center;
      overflow: hidden;
      .iconfont {
        font-size: 50px;
        color: #ddd;
      }
      img {
        width: 100%;
        height: 100%;
      }
    }
  }
  </style>