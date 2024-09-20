<template>
  <div>
   <el-row :gutter="100">
      <el-col :span="10"><div class="upload">Upload a log file (.txt)
        <el-row>
          <el-col :span="24"><div class="upload-file">
            <el-upload
              class="upload-demo"
              drag
              action=""
              :http-request="handleUpload"
              :before-upload="beforeUpload"
              :on-remove="handleRemove"
              multiple>
              <i class="el-icon-upload"></i>
              <div class="el-upload__text">Drag and drop files to here to upload, or<em> Click to upload</em></div>
              <div class="el-upload__tip" slot="tip">Only .txt file</div>
            </el-upload>
            </div></el-col>
        </el-row>
        </div></el-col>
      <el-col :span="12"><div class="result">Results:
        <el-row type=flex justify=center  v-if="file_result.size > 0">
          <el-tabs v-model="activeName" type="card">
            <el-tab-pane label="File-based" name="first"></el-tab-pane>
            <el-tab-pane label="User-based" name="second"></el-tab-pane>
          </el-tabs>
        </el-row>
      <el-row v-if="activeName === 'first'">
        <div v-for="(map2, filename) in file_result_arr" :key="filename">
          <h4>{{ filename }}</h4>
          <ol>
            <li v-for="(count, username) in map2" :key="username">
              {{ username }} - {{ count }} words
            </li>
          </ol>
        </div>
      </el-row>
      <el-row v-if="activeName === 'second'">
        <ol>
          <li v-for="([username, count]) in sortedUserCounts" :key="username">
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
  name: "HomeView",
  data() {
    return {
      fileContent:'',
      result: new Map(),
      file_result: new Map(),
      file_result_arr:[],
      activeName:'first',
      usernameCount: new Map(),
    }
  },
  computed: {
    sortedUserCounts() {
      return Array.from(this.usernameCount.entries()).sort((a, b) => b[1] - a[1]);
    },
  },
  methods: {
    handleUpload(params) {
      const file = params.file;
      const reader = new FileReader();

      reader.onload = (event) => {
        this.fileContent = event.target.result;

        let array = this.fileContent.split(/\s+/);
        let usernameCountMap = new Map();
        let currUser = ''
        for(let i = 0; i < array.length; i++) {
          if (array[i].startsWith('<') && array[i].endsWith('>')) {
            currUser = array[i]
          }
          else {
            if (currUser === '') {
              continue;
            }
            if (usernameCountMap.has(currUser)) {
              usernameCountMap.set(currUser, usernameCountMap.get(currUser) + 1);
          } else {
              usernameCountMap.set(currUser, 1);
          }

            if (this.usernameCount.has(currUser)) {
              this.usernameCount.set(currUser, this.usernameCount.get(currUser) + 1);
          } else {
              this.usernameCount.set(currUser, 1);
          }
          }
        }

        const sortedMapDescending = new Map(
          [...usernameCountMap.entries()].sort((a, b) => b[1] - a[1])
        );

        this.result = sortedMapDescending
        this.file_result.set(file.name, this.result)
        this.convertFileMap();
      };

      reader.readAsText(file);
    },
    beforeUpload(file) {
      const isTXT = file.type === 'text/plain';

      if (!isTXT) {
        this.$message.error('Only txt file');
        return false;
      }

      return true;
    },
    handleRemove(file) {
      this.file_result.delete(file.name)
      this.convertFileMap();
    },
    convertFileMap() {
      const convertedFileMap = Object.fromEntries(
          Array.from(this.file_result, ([filename, userMap]) => [filename, Object.fromEntries(userMap)])
        );

        this.file_result_arr = convertedFileMap;
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
  border: 5px orange solid;
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
