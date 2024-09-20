<template>
  <div>
   <el-row :gutter="100">
      <el-col :span="10"><div class="upload">Upload a log file (.txt)
        <el-row>
          <el-col :span="24"><div class="upload-file">
            <el-upload
              class="upload-demo"
              drag
              ref="upload"
              action=""
              :on-remove="handleRemove"
              :auto-upload="false"
              multiple>
              <i class="el-icon-upload"></i>
              <div class="el-upload__text">Drag and drop files to here to upload, or<em> Click to upload</em></div>
              <div class="el-upload__tip" slot="tip">Only .txt file</div>
            </el-upload>
            <el-button plain @click="handleUpload">Upload</el-button>
            </div></el-col>
        </el-row>
        </div></el-col>
      <el-col :span="12"><div class="result">Results:
      <el-row>
        <ol>
          <li v-for="([username, count]) in usernameCountArray" :key="username">
            {{ username }} - {{ count }} words
          </li>
        </ol>
      </el-row>
    </div></el-col>
  </el-row>
  </div>
</template>

<script>
export default {
  name: "AboutView",
  data() {
    return {
      usernameCount: new Map(),
    }
  },
  computed: {
      usernameCountArray() {
          return Array.from(this.usernameCount.entries()).sort((a, b) => b[1] - a[1]);
      }
  },
  methods: {
    handleUpload() {
      const uploadFiles = this.$refs.upload.uploadFiles;

      if (uploadFiles.length === 0) {
          this.$message.error('Please select a file to upload.');
          return;
      }

      for (const file of uploadFiles) {
          if (!file.name.endsWith('.txt')) {
              this.$message.error(`File "${file.name}" is not a .txt file.`);
              return; // Stop further processing
          }
      }

      const uploadPromises = uploadFiles.map(file => {
          return new Promise((resolve, reject) => {
              const reader = new FileReader();

              reader.onload = (e) => {
                  const content = e.target.result;
                  const formData = new FormData();
                  formData.append("chat_text", content);

                  this.$axios({
                      method: "post",
                      url: "/api/file/upload_text",
                      data: formData,
                  })
                  .then(response => {
                      resolve(response.data);
                  })
                  .catch(error => {
                      reject(error);
                  });
              };

              reader.onerror = () => {
                  reject(new Error('Error reading file: ' + file.name));
              };

              // Read the file as text
              reader.readAsText(file.raw);
          });
      });

      Promise.all(uploadPromises)
          .then(results => {
              this.$message.success('All files uploaded successfully');
              console.log(results);
              for (let i = 0; i < results.length; i++) {
                  const temp = results[i].data;
                  for (let j = 0; j < temp.length; j++) {
                      const user = temp[j];
                      const username = user[0];
                      const count = user[1];

                      if (this.usernameCount.has(username)) {
                          this.usernameCount.set(username, this.usernameCount.get(username) + count);
                      } else {
                          this.usernameCount.set(username, count);
                      }
                  }
              }
              console.log(this.usernameCount);
              this.usernameCount = new Map([...this.usernameCount]);
          })
          .catch(error => {
              this.$message.error('Error uploading files: ' + error.message);
          });
    },
  }
}
</script>

<style>
.upload {
  font-size: 36px;
  text-align: center;
  width: 400px;
  height: 300px;
}
.upload-file {
  margin-top: 10px;
}
.result {
  font-size: 30px;
  list-style-position: inside;
  text-align: center;
  width: 500px;
  height: 500px;
  border: 5px blue solid;
  overflow-y: auto;
}

.result::-webkit-scrollbar {
  width: 8px;
}

.result::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 4px;
}
</style>
