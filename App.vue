<template>
  <div class="dashboard">
    <!-- Summary Section -->
    <div class="summary-section">
      <p>Summary</p>
      <div class="summary-card">
        <div class="table-section">
          <table>
            <thead>
              <tr>
                <th>部门名称</th>
                <th>累计使用</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(count, department) in displayedDepartments" :key="department">
                <td>{{ department }}</td>
                <td>{{ count }}</td>
              </tr>
            </tbody>
          </table>
          <div class="pagination">
            <button @click="prevPage" :disabled="currentPage === 1">上一页</button>
            <button @click="nextPage" :disabled="currentPage >= maxPage">下一页</button>
          </div>
        </div>
        <div class="chart-section">
          <div ref="pieChart" class="chart"></div>
        </div>
      </div>
    </div>

    <!-- Summary Details Section -->
    <div class="summary-details-section">
      <p>Summary Details</p>
      <div class="service-container">
        <div class="service-buttons">
          <button
            v-for="service in serviceTypes"
            :key="service"
            @click="selectService(service)"
            class="service-button"
          >
            {{ service }}
          </button>
        </div>
        <hr />
        <div class="bar-chart-section">
          <div ref="barChart" class="chart"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as echarts from 'echarts';

export default {
  data() {
    return {
      data: null,
      displayedDepartments: {},
      serviceTypes: [],
      selectedService: null,
      currentPage: 1,
      topCount: 7, // 默认显示7条数据
    };
  },
  mounted() {
    this.fetchData();
  },
  methods: {
    async fetchData() {
      const response = await fetch('http://127.0.0.1:5000/api/user_center/get_summary_info/');
      this.data = await response.json();
      this.processData();
      this.initCharts();
    },
    processData() {
      this.serviceTypes = Object.keys(this.data.department_service_counts);
      this.selectedService = this.serviceTypes[0]; // 默认选择第一个服务
      this.updateTable(); // 确保更新表格数据
    },
    updateTable() {
      this.currentPage = 1; // 重置为第一页
      this.displayedDepartments = this.getPaginatedDepartments(); // 更新显示的部门数据
    },
    getPaginatedDepartments() {
      const departmentCounts = Object.entries(this.data.department_total_counts || {});
      const startIndex = (this.currentPage - 1) * this.topCount;
      const endIndex = startIndex + this.topCount;
      return Object.fromEntries(departmentCounts.slice(startIndex, endIndex));
    },
    nextPage() {
      if (this.currentPage < this.maxPage) {
        this.currentPage++;
        console.log(`当前页: ${this.currentPage}`); // 调试信息
        this.updateTable(); // 更新表格数据
      }
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
        console.log(`当前页: ${this.currentPage}`); // 调试信息
        this.updateTable(); // 更新表格数据
      }
    },
    get maxPage() {
      return this.data && this.data.department_total_counts
        ? Math.ceil(Object.keys(this.data.department_total_counts).length / this.topCount)
        : 1; // 如果没有数据，返回 1 页
    },
    selectService(service) {
      this.selectedService = service;
      this.updateTable(); // 更新表格以反映新服务的数据
      this.initCharts();
    },
    initCharts() {
      this.initPieChart();
      this.initBarChart();
    },
    initPieChart() {
      const pieChart = echarts.init(this.$refs.pieChart);
      const option = {
        title: {
          text: '服务占比',
          left: 'center',
        },
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b}: {c} ({d}%)',
        },
        series: [{
          name: '服务类型',
          type: 'pie',
          radius: '50%',
          data: this.data.service_counts.map(item => ({ name: item.type, value: item.count })),
          emphasis: {
            itemStyle: {
              shadowBlur: 10,
              shadowOffsetX: 0,
              shadowColor: 'rgba(0, 0, 0, 0.5)',
            },
          },
        }],
      };
      pieChart.setOption(option);
    },
    initBarChart() {
      const barChart = echarts.init(this.$refs.barChart);
      const departmentCounts = this.data.department_service_counts[this.selectedService] || {};

      const dynamicHeight = Math.max(400, Object.keys(departmentCounts).length * 30);
      this.$refs.barChart.style.height = `${dynamicHeight}px`;

      const option = {
        title: {
          text: `${this.selectedService}`,
        },
        tooltip: {
          trigger: 'item',
        },
        grid: {
          left: '10%',
          right: '10%',
          bottom: '30%',
          top: '30%',
        },
        xAxis: {
          type: 'category',
          data: Object.keys(departmentCounts),
          axisLabel: {
            rotate: 45,
            interval: 0,
          },
        },
        yAxis: {
          type: 'value',
        },
        series: [{
          data: Object.values(departmentCounts),
          type: 'bar',
          itemStyle: {
            color: (params) => {
              const colors = ['#FF6384', '#36A2EB', '#FFCE56', '#4BC0C0', '#9966FF', '#FF9F40'];
              return colors[params.dataIndex % colors.length];
            },
          },
        }],
        dataZoom: [
          {
            type: 'slider',
            show: true,
            xAxisIndex: [0],
            start: 0,
            end: 100,
            top: '10%',
          },
          {
            type: 'inside',
            xAxisIndex: [0],
            start: 0,
            end: 100,
          },
        ],
      };

      barChart.setOption(option);
    }
  },
};
</script>

<style scoped>
.dashboard {
  font-family: Arial, Helvetica, sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  background-color: beige;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.summary-section,
.summary-details-section {
  margin-bottom: 20px;
}

.summary-card {
  display: flex;
  justify-content: space-between;
  border: 2px solid #000; /* Enhance border color */
  border-radius: 8px;
  overflow: hidden;
}

.table-section {
  width: 50%;
  padding: 10px;
}

.table-section table {
  width: 100%;
  border-collapse: collapse;
}

.table-section th,
.table-section td {
  border: 1px solid #e0e0e0;
  text-align: left;
  padding: 8px;
}

.pagination {
  display: flex;
  justify-content: space-between;
  margin-top: 10px;
}

.chart-section {
  width: 50%;
  padding: 10px;
}

.service-container {
  border: 2px solid #000; /* Enhance border color */
  border-radius: 8px;
  padding: 10px;
}

.service-buttons {
  display: flex;
  gap: 2px;
  margin-bottom: 10px;
}

.service-button {
  border: 1px solid black;
  background-color: white;
  color: black;
  padding: 5px 10px;
  border-radius: 5px;
  transition: background-color 0.3s, color 0.3s;
}

.service-button:hover {
  background-color: black;
  color: white;
}

.bar-chart-section {
  height: 400px;
}

.chart {
  height: 400px;
}

/* 对齐修改 */
.summary-section p,
.summary-details-section p {
  margin: 0; 
  text-align: left;
  margin-bottom: 20px;
  font-size: 24px;
}
</style>
