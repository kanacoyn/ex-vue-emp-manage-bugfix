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

    <form>
      <input id="searchName" type="text" v-model="searchName" />
      <label for="searchName">検索したい名前(部分一致検索)</label>
      <div class="error">{{ searchNameError }}</div>
      <div class="searchBtn">
        <button
          class="btn btn-register waves-effect waves-light"
          type="button"
          v-on:click="onSearchClick"
        >
          検索する
        </button>
      </div>
    </form>
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
            <td>{{ employee.formatHireDate }}</td>
            <td>{{ employee.dependentsCount }}人</td>
          </tr>
        </tbody>
      </table>
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
  // 名前検索
  private searchName = "";
  // 名前検索していなかった場合
  private searchNameError = "";

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
  }
  /**
   * 現在表示されている従業員一覧の数を返す.
   *
   * @returns 現在表示されている従業員一覧の数
   */
  get getEmployeeCount(): number {
    return this.currentEmployeeList.length;
  }

  /**
   * あいまい検索をする.
   *
   * @remarks
   * Vuexストアから検索文字列に一致したものを絞り込んで出力
   * ない場合は「１件もありませんでしたので全件表示します」と出力して全件検索を行います。
   */
  onSearchClick(): void {
    // 入力された文字列で絞り込みを行う
    this.currentEmployeeList = this.$store.getters.getSearchEmployeeByName(
      this.searchName
    );
    this.searchNameError = "";

    // 検索されない場合は、メッセージを出して全件検索を行う
    if (this.currentEmployeeList.length === 0 || this.searchName === "") {
      this.searchNameError = "１件もありませんでしたので全件検索します";
      this.currentEmployeeList = this.$store.getters.getAllEmployees;
      this.searchName = "";
    }
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
</style>
