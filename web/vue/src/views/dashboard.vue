<template>
  <el-row :gutter="12">
    <el-col :span="6" v-loading="loading">
      <el-card shadow="hover">
        <div class="flex justify-between">
          <div class="icon">
            <el-icon>
              <AlarmClock/>
            </el-icon>
          </div>
          <div class="data flex justify-between">
            <div>任务总数</div>
            <span>{{ totalGroup.taskCount }}</span>
          </div>
        </div>
      </el-card>
    </el-col>
    <el-col :span="6" v-loading="loading">
      <el-card shadow="hover">
        <div class="flex justify-between">
          <div class="icon">
            <el-icon>
              <House/>
            </el-icon>
          </div>
          <div class="data flex justify-between">
            <div>进程总数</div>
            <span>{{ totalGroup.processCount }}</span>
          </div>
        </div>
      </el-card>
    </el-col>
    <el-col :span="6" v-loading="loading">
      <el-card shadow="hover">
        <div class="flex justify-between">
          <div class="icon">
            <el-icon>
              <User/>
            </el-icon>
          </div>
          <div class="data flex justify-between">
            <div>用户总数</div>
            <span>{{ totalGroup.userCount }}</span>
          </div>
        </div>
      </el-card>
    </el-col>
    <el-col :span="6" v-loading="loading">
      <el-card shadow="hover">
        <div class="flex justify-between">
          <div class="icon">
            <el-icon>
              <FolderAdd/>
            </el-icon>
          </div>
          <div class="data flex justify-between">
            <div>项目总数</div>
            <span>{{ totalGroup.projectCount }}</span>
          </div>
        </div>
      </el-card>
    </el-col>
  </el-row>
  <el-row style="margin-top: 15px" :gutter="12">
    <el-col :span="10" v-loading="loading">
      <el-card shadow="hover">
        <div id="project-tasks" style="height: 300px"></div>
      </el-card>
    </el-col>
    <el-col :span="14" v-loading="loading">
      <el-card shadow="hover">
        <div id="users-chart" style="height: 300px"></div>
      </el-card>
    </el-col>
  </el-row>
  <el-row style="margin-top: 15px" :gutter="12">
    <el-col :span="24" v-loading="loading">
      <el-card shadow="hover">
        <div id="project-new-data" style="height: 400px"></div>
      </el-card>
    </el-col>
  </el-row>
</template>

<script>
import * as echarts from 'echarts';

import dashboardService from '../api/dashboard'

export default {
  name: 'dashboard-view',
  data() {
    return {
      loading: true,
      totalGroup: {taskCount: 0, processCount: 0, userCount: 0, projectCount: 0},
      projectTaskChart: null,
      projectNewX: [],
      projectNewCharts: [],
      activeUsers: [],
      projectTasks: [],
      projectNewChart: null,
    }
  },
  beforeUnmount() {
    //https://blog.csdn.net/qq_37656005/article/details/119904510 切换页面之前销毁实例,避免之后切换到该页面图表不显示
    if (this.projectTaskChart) {
      this.projectTaskChart.dispose()
    }
    if (this.userChart) {
      this.userChart.dispose()
    }
    if (this.projectNewChart) {
      this.projectNewChart.dispose()
    }
  },
  mounted() {
    let _this = this
    console.log('mounted')
    dashboardService.get({}, function (data) {
      _this.loading = false;
      _this.totalGroup = data.totalGroup
      let taskChartContainer = document.getElementById('project-tasks');
      let projectTaskChart = echarts.init(taskChartContainer);
      //https://blog.csdn.net/weixin_53545517/article/details/120865314 处理echarts在vue中切换到其他页面再返回来就不显示的问题
      taskChartContainer.setAttribute('_echarts_instance_', '')
      projectTaskChart.setOption({
        title: {
          text: '项目任务数',
          // subtext: '各项目下的任务数量',
          left: 'center'
        },
        tooltip: {
          trigger: 'item'
        },
        legend: {
          orient: 'vertical',
          left: 'left'
        },
        series: [
          {
            type: 'pie',
            radius: '50%',
            // left:'30%',
            data: data.projectTasks,
            emphasis: {
              itemStyle: {
                shadowBlur: 10,
                shadowOffsetX: 0,
                shadowColor: 'rgba(0, 0, 0, 0.5)'
              }
            }
          }
        ]
      },true);
      _this.projectTaskChart = projectTaskChart

      _this.userChart = echarts.init(document.getElementById('users-chart'));
      let userX = [], userY = []
      data.activeUsers.forEach(user => {
        userX.push(user.username)
        userY.push(user.count)
      })
      _this.userChart.setOption({
        title: {
          text: '30天活跃用户'
        },
        tooltip: {
          trigger: 'item'
        },
        xAxis: {
          type: 'category',
          data: userX
        },
        yAxis: {
          type: 'value'
        },
        series: [
          {
            data: userY,
            type: 'bar'
          }
        ]
      });

      //参考 https://blog.51cto.com/u_15127614/3928739 修改2
      let projectNewChart = echarts.init(document.getElementById('project-new-data'));
      projectNewChart.setOption({
        title: {
          text: '14周内系统新增数据'
        },
        tooltip: {
          trigger: 'axis'
        },
        legend: {
          // data: ['ims新增任务', 'ims新增进程', 'oa新增进程']
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        toolbox: {
          feature: {
            saveAsImage: {}
          }
        },
        xAxis: {
          type: 'category',
          boundaryGap: false,
          data: data.projectNewX
        },
        yAxis: {
          type: 'value'
        },
        series: data.projectNewCharts
      },true)
      _this.projectTaskChart = projectNewChart
    })
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.flex {
  display: flex;
}

.justify-between {
  justify-content: space-between;
}

.icon {
  padding: 16px;
}

.icon .el-icon {
  font-size: 40px;
}

.data {
  flex-direction: column;
}

.data > span {
  text-align: right;
  font-size: 20px;
  line-height: 1;
  font-weight: 700;
}
</style>
