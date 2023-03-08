<template>
  <div class="upload select">
    <div v-if="uploadStatus === 1" class="upload-select" @click="selectFile" />
    <div
      v-else-if="uploadStatus === 2"
      class="upload-progress"
      style="--percent: 0"
      ref="progressRef"
    >
      <div class="progress-bar">
        <button>取消</button>
      </div>
    </div>
    <div v-else-if="uploadStatus === 3" class="upload-result">
      <button>X</button>
    </div>
    <img v-show="uploadStatus !== 1" :src="previewUrl" alt="" class="preview" ref="imgRef" />
    <input type="file" ref="fileRef" />
  </div>
</template>

<script setup>
// 1:初始状态；2：上传中；3：上传结束
let uploadStatus = $ref(1)
let previewUrl = $ref('')
const fileRef = $ref()
const progressRef = $ref()
const imgRef = $ref()
function selectFile() {
  fileRef.click()
  fileRef.onchange = () => {
    // 读到选中后文件的file对象[本质是blob对象]
    const file = fileRef.files[0]
    const reader = new FileReader() // 文件读取器
    reader.onload = (e) => {
      imgRef.src = e.target.result
    }
    reader.readAsDataURL(file) // 读取选中的文件，生成可以访问的data:url(base64编码)
    // 开始发送请求
    uploadStatus = 2
    const xhr = new XMLHttpRequest()
    xhr.addEventListener('load', () => {
      // 上传完成后触发
      const resp = JSON.parse(xhr.responseText).data
      previewUrl.value = resp
      uploadStatus = 3
    })
    xhr.addEventListener('progress', (e) => {
      // 上传中触发
      const percent = Math.floor((e.loaded / e.total) * 100)
      progressRef.style.setProperty('--percent', percent)
    })
    xhr.open('POST', 'http://127.0.0.1:3000/api/upload/base64')
    // 自动构建multipart/form-data格式的消息体
    const formData = new FormData()
    formData.append('avatar', file)
    // 发送请求
    xhr.send(formData)
  }
}
</script>

<style scoped>
.upload {
  --border-color: #dcdfe6;
  --font-color: #8c939d;
  --primary-color: #409eff;
  --danger-color: #eb685e;
}
.upload {
  width: 150px;
  height: 150px;
  position: relative;
  overflow: hidden;
  border-radius: 5px;
}
.upload > * {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  z-index: 2;
}
.upload button {
  border: none;
  outline: none;
  background: none;
  color: inherit;
  font-size: inherit;
  cursor: pointer;
  user-select: none;
}
.upload-select {
  border-radius: inherit;
  border: 1px dashed var(--border-color);
  cursor: pointer;
}
.upload-select::before,
.upload-select::after {
  content: '';
  position: absolute;
  left: 50%;
  top: 50%;
  width: 30px;
  height: 3px;
  border-radius: 3px;
  background: var(--font-color);
  transform: translate(-50%, -50%);
}
.upload-select::after {
  transform: translate(-50%, -50%) rotate(90deg);
}
.upload-select:hover {
  border-color: var(--primary-color);
}
.upload-progress {
  background: #00000080;
}
.upload-progress::after {
  content: '文件上传中...';
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, 5px);
  white-space: nowrap;
  opacity: 0.8;
  color: #fff;
  font-size: 12px;
}
.progress-bar {
  position: absolute;
  width: 90%;
  height: 3px;
  background: #fff;
  border-radius: 3px;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  color: #fff;
  font-size: 12px;
}
.progress-bar::before {
  counter-reset: progress var(--percent);
  content: counter(progress) '%';
  position: absolute;
  left: 50%;
  top: -20px;
  transform: translateX(-50%);
}
.progress-bar::after {
  content: '';
  position: absolute;
  left: 0;
  border-radius: inherit;
  width: calc(1% * var(--percent));
  height: 100%;
  background: var(--primary-color);
}
.progress-bar button {
  left: 50%;
  position: absolute;
  top: 25px;
  transform: translateX(-50%);
}
.progress-bar button:hover {
  color: var(--danger-color);
}
.upload-result {
  border: 1px dashed var(--border-color);
  border-radius: inherit;
  overflow: hidden;
}
.upload-result button {
  width: 30px;
  height: 20px;
  background: var(--font-color);
  position: absolute;
  right: 0;
  top: 0;
  border-radius: 2px;
  color: #fff;
}
.upload-result button:hover {
  background: var(--danger-color);
}
.upload .preview {
  object-fit: contain;
  z-index: 1;
}
.upload input {
  display: none;
}
</style>
