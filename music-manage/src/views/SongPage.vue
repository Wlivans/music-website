<template>
  <div class="table">
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-tickets"></i> 歌曲信息
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <div class="handle-box">
        <el-button type="primary" size="small" class="handle-del mr10" @click="delAll">批量删除</el-button>
        <el-input v-model="select_word" size="small" placeholder="筛选关键词" class="handle-input mr10"></el-input>
        <el-button type="primary" size="small" @click="centerDialogVisible = true">添加歌曲</el-button>
      </div>
      <el-table :data="data" size="small" border style="width: 100%" ref="multipleTable" height="550px" @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="40"></el-table-column>
        <el-table-column label="歌手图片" width="110" align="center">
          <template v-slot="scope">
            <div style="width: 80px; height: 80px; overflow: hidden">
              <img :src="getUrl(scope.row.pic)" alt="" style="width: 100%;"/>
            </div>
            <div class="play" @click="setSongUrl(scope.row.url, scope.row.name)">
              <div v-if="toggle !== scope.row.name">
                <svg class="icon" aria-hidden="true">
                  <use :xlink:href="BOFANG"></use>
                </svg>
              </div>
              <div v-if="toggle === scope.row.name">
                <svg class="icon" aria-hidden="true">
                  <use :xlink:href="playIcon"></use>
                </svg>
              </div>
            </div>
          </template>
        </el-table-column>
        <el-table-column label="歌名" prop="name" width="150" align="center"></el-table-column>
        <el-table-column label="专辑" prop="introduction" width="150" align="center"></el-table-column>
        <el-table-column label="歌词" align="center">
          <template v-slot="scope">
            <ul style="height: 100px; overflow: scroll">
              <li v-for="(item, index) in parseLyric(scope.row.lyric)" :key="index">
                {{ item}}
              </li>
            </ul>
          </template>
        </el-table-column>
        <el-table-column label="资源更新" width="100" align="center">
          <template v-slot="scope">
            <el-upload
              class="upload-demo"
              :action="uploadUrl(scope.row.id)"
              :show-file-list="false"
              :on-success="handleAvatarSuccess"
              :before-upload="beforeAvatarUpload"
              >
                <el-button size="small">更新图片</el-button>
            </el-upload>
            <br>
            <el-upload
              class="upload-demo change"
              :action="uploadSongUrl(scope.row.id)"
              :show-file-list="false"
              :on-success="handleSongSuccess"
              :before-upload="beforeSongUpload">
              <el-button size="small">更新歌曲</el-button>
            </el-upload>
          </template>
        </el-table-column>
        <el-table-column label="评论" width="80" align="center">
            <template  v-slot="scope">
                <el-button size="small" @click="getComment(data[scope.$index].id)">评论</el-button>
            </template>
        </el-table-column>
        <el-table-column label="操作" width="150" align="center">
            <template v-slot="scope">
                <el-button size="small" @click="handleEdit(scope.row)">编辑</el-button>
                <el-button size="small" type="danger" @click="handleDelete(scope.row.id)">删除</el-button>
            </template>
        </el-table-column>
      </el-table>
      <div class="pagination">
        <el-pagination
          @current-change="handleCurrentChange"
          background
          layout="total, prev, pager, next"
          :current-page="currentPage"
          :page-size="pageSize"
          :total="tableData.length">
        </el-pagination>
      </div>
    </div>

    <!--添加歌曲-->
    <el-dialog title="添加歌曲" v-model="centerDialogVisible" width="400px" center>
      <el-form action="" :model="registerForm" id="tf">
        <div>
          <label>歌曲名</label>
          <el-input type="text" name="name"></el-input>
        </div>
        <div>
          <label>专辑</label>
          <el-input type="text" value="" name="introduction"></el-input>
        </div>
        <div>
          <label>歌词</label>
          <el-input type="textarea" value="" name="lyric"></el-input>
        </div>
        <div>
          <label>歌曲上传</label>
          <br>
          <input type="file" name="file" id="upadte-file-input">
        </div>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="centerDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addSong">确 定</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 编辑弹出框 -->
    <el-dialog title="编辑" v-model="editVisible" width="400px">
      <el-form ref="form" :model="form" label-width="80px">
        <el-form-item label="歌手-歌曲" size="small">
          <el-input v-model="form.name"></el-input>
        </el-form-item>
        <el-form-item label="简介" size="small">
          <el-input v-model="form.introduction"></el-input>
        </el-form-item>
        <el-form-item label="歌词" size="small">
          <el-input  type="textarea" v-model="form.lyric"></el-input>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button size="small" @click="editVisible = false">取 消</el-button>
          <el-button type="primary" size="small" @click="saveEdit">确 定</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 删除提示框 -->
    <yin-del-dialog :delVisible="delVisible" @deleteRow="deleteRow" @cancelRow="delVisible = $event"></yin-del-dialog>
  </div>
</template>

<script>
import { mixin } from '../mixins'
import { mapGetters } from 'vuex'
import { ICON, COMMENT } from '../enums'
import { HttpManager } from '../api/index'
import YinDelDialog from '@/components/dialog/YinDelDialog'

export default {
  name: 'SongPage',
  mixins: [mixin],
  components: {
    YinDelDialog
  },
  data () {
    return {
      toggle: false, // 控制播放图标状态
      singerId: '',
      singerName: '',
      registerForm: {
        name: '',
        singerName: '',
        introduction: '',
        lyric: ''
      },
      tableData: [],
      tempDate: [],
      is_search: false,
      multipleSelection: [], // 记录要删除的歌曲
      centerDialogVisible: false,
      editVisible: false,
      delVisible: false,
      select_word: '',
      form: {
        id: '',
        singerId: '',
        name: '',
        introduction: '',
        createTime: '',
        updateTime: '',
        pic: '',
        lyric: '',
        url: ''
      },
      pageSize: 5, // 页数
      currentPage: 1, // 当前页
      idx: -1,
      BOFANG: ICON.BOFANG,
      ZANTING: ICON.ZANTING
    }
  },
  computed: {
    ...mapGetters([
      'isPlay' // 播放状态
    ]),
    // 计算当前表格中的数据
    data () {
      return this.tableData.slice((this.currentPage - 1) * this.pageSize, this.currentPage * this.pageSize)
    },
    playIcon () {
      if (this.isPlay) {
        return this.ZANTING
      } else {
        return this.BOFANG
      }
    }
  },
  watch: {
    select_word: function () {
      if (this.select_word === '') {
        this.tableData = this.tempDate
      } else {
        this.tableData = []
        for (let item of this.tempDate) {
          if (item.name.includes(this.select_word)) {
            this.tableData.push(item)
          }
        }
      }
    }
  },
  created () {
    this.singerId = this.$route.query.id
    this.singerName = this.$route.query.name
    this.getData()
  },
  unmounted () {
    this.$store.commit('setIsPlay', false)
  },
  methods: {
    // 获取歌曲
    getData () {
      this.tableData = []
      this.tempDate = []
      HttpManager.getSongOfSingerId(this.singerId).then((res) => {
        this.tableData = res
        this.tempDate = res
        this.currentPage = 1
      }).catch(err => {
        console.error(err)
      })
    },
    setSongUrl (url, name) {
      this.$store.commit('setUrl', this.$store.state.HOST + url)
      this.toggle = name
      if (this.isPlay) {
        this.$store.commit('setIsPlay', false)
      } else {
        this.$store.commit('setIsPlay', true)
      }
    },
    // 更新歌曲图片
    uploadUrl (id) {
      return `${this.$store.state.HOST}/song/images/update?id=${id}`
    },
    // 更新歌曲url
    uploadSongUrl (id) {
      return `${this.$store.state.HOST}/song/url/update?id=${id}`
    },
    beforeSongUpload (file) {
      var testmsg = file.name.substring(file.name.lastIndexOf('.') + 1)
      const extension = testmsg === 'mp3'
      if (!extension) {
        this.$message({
          message: '上传文件只能是mp3格式！',
          type: 'error'
        })
      }
      return extension
    },
    // 获取当前页
    handleCurrentChange (val) {
      this.currentPage = val
    },
    handleSongSuccess (res) {
      if (res.code === 1) {
        this.getData()
        this.$notify({
          title: '上传成功',
          type: 'success'
        })
      } else {
        this.$notify({
          title: '上传失败',
          type: 'error'
        })
      }
    },
    // 添加音乐
    addSong () {
      var form = new FormData(document.getElementById('tf'))
      form.append('singerId', this.singerId)
      form.set('name', this.singerName + '-' + form.get('name'))
      if (!form.get('lyric')) {
        form.set('lyric', '[00:00:00]暂无歌词')
      }
      var req = new XMLHttpRequest()
      req.onreadystatechange = () => {
        if (req.readyState === 4 && req.status === 200) {
          let res = JSON.parse(req.response)
          if (res.code) {
            this.getData()
            this.registerForm = {}
            this.$notify({
              title: '添加成功',
              type: 'success'
            })
          } else if (!res.code) {
            this.$notify({
              title: '添加失败',
              type: 'error'
            })
          }
        }
      }
      req.open('post', `${this.$store.state.HOST}/song/add`, false)
      req.send(form)
      this.centerDialogVisible = false
    },
    // 编辑
    handleEdit (row) {
      this.idx = row.id
      this.form = {
        id: row.id,
        singerId: row.singerId,
        name: row.name,
        introduction: row.introduction,
        createTime: row.createTime,
        updateTime: row.updateTime,
        pic: row.pic,
        lyric: row.lyric,
        url: row.url
      }
      this.editVisible = true
    },
    getComment (id) {
      this.routerManager(COMMENT, { path: COMMENT, query: { id, type: 0 } })
    },
    // 保存编辑
    saveEdit () {
      let params = new URLSearchParams()
      params.append('id', this.form.id)
      params.append('singerId', this.form.singerId)
      params.append('name', this.form.name)
      params.append('introduction', this.form.introduction)
      params.append('lyric', this.form.lyric)
      HttpManager.updateSongMsg(params)
        .then(res => {
          if (res) {
            this.getData()
            this.$notify({
              title: '编辑成功',
              type: 'success'
            })
          } else {
            this.$notify({
              title: '删除失败',
              type: 'error'
            })
          }
        })
        .catch(err => {
          console.error(err)
        })
      this.editVisible = false
    },
    // 确定删除
    deleteRow () {
      HttpManager.deleteSong(this.idx)
        .then(response => {
          if (response) {
            this.getData()
            this.$notify({
              title: '删除成功',
              type: 'success'
            })
          } else {
            this.$notify({
              title: '删除失败',
              type: 'error'
            })
          }
        })
        .catch(err => {
          console.error(err)
        })
      this.delVisible = false
    },
    // 解析歌词
    parseLyric (text) {
      let lines = text.split('\n')
      let pattern = /\[\d{2}:\d{2}.(\d{3}|\d{2})\]/g
      let result = []

      // 对于歌词格式不对的特殊处理
      if (!(/\[.+\]/.test(text))) {
        return [text]
      }
      for (let item of lines) {
        if (pattern.test(item)) {
          let value = item.replace(pattern, '') // 存歌词
          result.push(value)
        }
      }
      return result
    }
  }
}

</script>

<style scoped>
.handle-box {
    margin-bottom: 20px;
}
.handle-input {
    width: 300px;
    display: inline-block;
}
.el-input__inner{
  background-color: aqua
}
.play {
    position: absolute;
    z-index: 100;
    width: 80px;
    height: 80px;
    top: 18px;
    left: 15px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
}
.icon {
  width: 2em;
  height: 2em;
  color: white;
  fill: currentColor;
  overflow: hidden;
}
.pagination {
    display: flex;
    justify-content: center;
}
</style>
