<template>
  <div class="board-view">
    <div class="board-task-view">
      <div class="board-warnings"></div>
      <draggable
        class="board-task-stages horizontal-scroll move-able"
        v-model="taskStages"
        :options="stageDragOptions">
        <board-view-stage
          :project="project"
          :drag-options="taskDragOptions"
          v-for="stage in taskStages"
          :key="stage._id"
          :stage="stage">
        </board-view-stage>
        <div slot="footer" class="task-stage-creator">
          <div class="creator-warp" v-if="!newStage.show" @click="newStage.show=true">
            <span class="ion ion-normal ion-plus mr-1"></span><span>新建任务阶段</span>
          </div>
          <div class="creator-warp creator-form-warp" v-if="newStage.show">
            <div>
              <input type="text"
                     v-model="newStage.name"
                     placeholder="新建任务阶段..."
                     class="form-control stage-name">
            </div>
            <div class="action-btn-warp float-right">
              <a @click="newStage.show=false" class="btn btn-link contract-creator-handler">取消</a>
              <a @click="createStage" class="btn btn-primary submit-add">保存</a>
            </div>
          </div>
        </div>
      </draggable>
    </div>
    <!--详情面板-->
    <keep-alive>
      <router-view></router-view>
    </keep-alive>
  </div>
</template>

<script>
  import Api from '@/utils/api'

  import Draggable from 'vuedraggable'
  import BoardViewStage from '../../components/BoardView/BoardViewStage'

  export default {
    name: 'ProjectTask',
    components: {
      Draggable, BoardViewStage
    },
    computed: {
      //这个阶段是Vuex存储
      taskStages: {
        get() {
          return this.$store.state.taskStages;
        },
        set(stages) {
          this.$store.dispatch('setTaskStages', stages);
        }
      },
      project() {
        return this.$store.state.project;
      }
    },
    data() {
      return {
        taskList: {},
        newStage: {
          name: '',
          show: false
        },
        taskDragOptions: {
          group: 'task',
          draggable: '.task-item',
          ghostClass: 'ghost'
        },
        stageDragOptions: {
          group: 'stage',
          draggable: '.task-stage'
        }
      }
    },
    methods: {
      //为任务列表创建一个新的阶段
      async createStage() {
        let stage = {
          '_projectId': this.project._id,
          '_taskListId': this.taskList._id,
          'name': this.newStage.name,
          'title': this.newStage.name,
          'sort': this.taskStages.length
        };
        let res = await Api.createTaskStage(stage);
        this.newStage.name = '';
        this.$store.dispatch('addTaskStages', res);
      }
    },
    async created() {
      this.taskList._id = this.$route.params._taskListId;
    }
  }

</script>

<style scoped lang="scss">

  .task-stage-creator {
    .creator-warp {
      cursor: pointer;
      line-height: 30px;
      padding: 10px;
      color: #a6a6a6;
    }
    .creator-form-warp {
      padding: 10px;
      input.stage-name {
        padding: 10px;
        border-radius: 0;
        border: none;
        line-height: 20px;
        margin-bottom: 5px;
        &::placeholder {
          color: #a6a6a6;
        }
      }
      .action-btn-warp {
        padding: 5px;
      }
      .submit-add {
        color: #fff;
      }
    }
  }
</style>
