<template>
  <div class="Wrap full">
    <div class="View-Wrap full flex-column">
      <div class="select-wrap">
        <div>
          <stationSelect
            :defaulState="false"
            :emitState="true"
            @selectedStation="onSelectedPC"
            :optindata="optinedataPC"
            :defaultKey="optinePcKey"
            selName="批次"
            ref="StationSelect"
          ></stationSelect>
          <stationSelect
            :defaulState="false"
            :emitState="true"
            @selectedStation="onSelectedJD"
            :optindata="optinedataJD"
            :defaultKey="optineJdKey"
            selName="基地"
            ref="StationSelect"
          ></stationSelect>
          <stationSelect
            :defaulState="false"
            :emitState="true"
            @selectedStation="selectedStation"
            :optindata="optinedataDZ"
            selName="电站"
            ref="selectedStation"
          ></stationSelect>
          <!-- <station ref="selectedStation" :defaulState="true" :emitState="true" @selectedStation="selectedStation"></station> -->
        </div>
      </div>
      <div class="table-box flex-1 formcls" style="margin-top:20px">
        <el-table
          :data="tableData"
          class="table"
          stripe
          highlight-current-row
          @current-change="handleCurrentChange"
          style="width: 100%; "
          size="medium"
          border
        >
          <el-table-column
            type="index"
            label="序号"
            width="80px"
            align="center"
          >
          </el-table-column>
          <el-table-column
            property="batch"
            label="批次名称"
            show-overflow-tooltip
            align="center"
          >
          </el-table-column>
          <el-table-column
            property="baseName"
            label="基地名称"
            show-overflow-tooltip
            align="center"
          >
          </el-table-column>
          <el-table-column
            property="projectname"
            label="电站名称"
            show-overflow-tooltip
            align="center"
          >
          </el-table-column>
          <el-table-column
            property="modelcode"
            label="组件名称"
            show-overflow-tooltip
            align="center"
          >
          </el-table-column>
          <el-table-column
            property="checktime"
            label="检查时间"
            show-overflow-tooltip
            align="center"
          >
            <span slot-scope="scope">{{
              $moment(scope.row.checktime).format("YYYY-MM-DD")
            }}</span>
          </el-table-column>
          <el-table-column
            property="createTime"
            label="填报时间"
            show-overflow-tooltip
            align="center"
          >
            <span slot-scope="scope">{{
              $moment(scope.row.createtime).format("YYYY-MM-DD HH:mm:ss")
            }}</span>
          </el-table-column>
          <el-table-column label="操作" width="150" align="center">
            <template slot-scope="scope">
              <span class="op-btn">
                <img
                  src="/img/icon05.png"
                  alt=""
                  @click="detailForm(scope.row)"
                />
              </span>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div class="m-t-20">
        <pageBox :totalNum="totalNum" @Refresh="Refresh" ref="pages"></pageBox>
      </div>
    </div>
    <component :is="dialogTypeName" :detailData="selectedRow"></component>
  </div>
</template>
<script>
import From from "./From";
import stationSelect from "@/components/station";
import pageBox from "@/components/Page1.vue";
export default {
  components: { From, pageBox, stationSelect },
  data() {
    return {
      optinedataPC: [], //批次数组
      optinedataJD: [], //基地数组
      optinedataDZ: [], //电站数组
      optinePcKey: {
        value: "name",
        label: "id"
      },
      optineJdKey: {
        value: "id",
        label: "name"
      },
      dialogTypeName: null,
      tableData: [
        { proName: "项目xxxx", compName: "100MW", createTime: "2018-05-21" }
      ],
      projectList: [
        { label: "项目1", value: "项目1" },
        { label: "项目2", value: "项目2" }
      ],
      pageInfo: {
        page: 1,
        pageSize: 20
      },
      form: {
        projectcode: "",
        page: 1,
        pageSize: 20
      },
      totalNum: 0,
      flag: true,
      selectedRow: {}
    };
  },
  mounted() {
    this.getSelateData();
    this.Refresh()
  },
  methods: {
    //获取selsct数据
    getSelateData() {
      this.$fetch("/MtrBase/GetBatchBaseTree").then(res => {
        this.optinedataPC = res.data;
        this.optinedataJD = res.data[0].childTreeNode;
        this.onSelectedJD(res.data[0].childTreeNode[0].id);
      });
    },
    onSelectedPC(data) {
      this.optinedataJD = [];
      this.optinedataPC.map(item => {
        if (item.name == data) {
          this.optinedataJD = item.childTreeNode;
        }
      });
    },
    onSelectedJD(data) {
      this.optinedataDZ = [];
      this.$fetch("/RealTimeData/GetProList?baseid=" + data).then(res => {
        this.optinedataDZ = res.data;
      });
    },
    detailForm(data) {
      this.selectedRow = data;
      this.dialogTypeName = "From";
    },
    handleCurrentChange() {},
    Refresh() {
      this.queryData(this.pageInfo.page);
    },
    selectedStation() {
      //控制下拉不刷新表格
      if (this.flag) this.queryData(1);
    },
    queryData(page) {
      this.form.page = page ? page : 1;
      this.pageInfo.page = page ? page : 1;
      this.form.projectcode = this.$refs.selectedStation.stationid;
      this.$refs["pages"].pageInfo = this.pageInfo;
      this.$fetch( "/BisComponentacceptance/QueryData",this.form).then(res => {
        this.tableData = res.data.data;
        this.totalNum = res.data.totalCount;
      });
    }
  }
};
</script>
<style lang="scss" scoped>
.select-wrap {
  display: flex;
  & > div {
    margin-right: 20px;
  }
}
</style>
