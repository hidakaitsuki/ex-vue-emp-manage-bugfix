<template>
  <div class="container">
    <!-- パンくずリスト -->
    <nav>
      <div class="nav-wrapper">
        <div class="col s12 teal">
          <a class="breadcrumb">従業員リスト</a>
        </div>
      </div>
    </nav>
    <div>従業員数:{{ getEmployeeCount }}人</div>
    <div class="row">
      <table class="striped">
        <thead>
          <tr>
            <th>名前</th>
            <th>入社日</th>
            <th>扶養人数</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="employee of currentEmployeeList" v-bind:key="employee.id">
            <td>
              <router-link :to="'/employeeDetail/' + employee.id">{{
                employee.name
              }}</router-link>
            </td>
            <td>{{ employee.formathiredate }}</td>
            <td>{{ employee.dependentsCount }}人</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div id="pagebutton">
      <button
        type="button"
        v-for="page of pageArray"
        v-bind:key="page"
        @click="onPageBottunclick(page)"
      >
        {{ page }}
      </button>
    </div>
  </div>
</template>
<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { Employee } from "@/types/employee";

/**
 * 従業員一覧を表示する画面.
 */
@Component
export default class EmployeeList extends Vue {
  // 従業員一覧
  private currentEmployeeList: Array<Employee> = [];
  // 従業員数
  private employeeCount = 0;
  // ページ番号
  private pageArray: Array<number> = [];
  private displayEmployee: Array<Employee> = [];
  /**
   * Vuexストアのアクション経由で非同期でWebAPIから従業員一覧を取得する.
   *
   * @remarks
   * Vueインスタンスが生成されたタイミングで
   * Vuexストア内のアクションを呼ぶ(ディスパッチする)。
   * ライフサイクルフックのcreatedイベント利用。
   *
   * 取得してからゲットするため、async awaitを利用している。
   */
  async created(): Promise<void> {
    await this.$store.dispatch("getEmployeeList");

    // 従業員一覧情報をVuexストアから取得
    // 非同期で外部APIから取得しているので、async/await使わないとGetterで取得できない
    // ページング機能実装のため最初の10件に絞り込み
    this.currentEmployeeList = this.$store.getters.getAllEmployees;

    //ページ数をボタンで表示する
    let pagemaxnumber = Math.floor(this.currentEmployeeList.length / 10 + 1);
    if (this.currentEmployeeList.length % 10 == 0) {
      pagemaxnumber -= 1;
    }
    for (let i = 1; i <= pagemaxnumber; i++) {
      this.pageArray.push(i);
    }
// 初期の従業員一覧
    for (let i = 1; i <= 10; i++) {
      this.displayEmployee.push(this.currentEmployeeList[i]);
    }
    this.currentEmployeeList = this.displayEmployee;
  }
  /**
   * 現在表示されている従業員一覧の数を返す.
   *
   * @returns 全ての従業員一覧の数
   */
  get getEmployeeCount(): number {
    return this["$store"].getters.getAllEmployees.length;
  }
  /**
   * ページ数のボタンを押したときに、10件ずつ表示する.
   */
  onPageBottunclick(page: number): void {
    this.currentEmployeeList = this["$store"].getters.getAllEmployees.slice(
      page * 10 - 10,
      page * 10
    );
  }
}
</script>

<style scoped>
.searchForm {
  margin-bottom: 20px;
  width: 450px;
  margin: 0 auto;
}

.searchBtn {
  display: block;
  width: 150px;
  margin: 0 auto;
}
#pagebutton {
  text-align: center;
}
</style>
