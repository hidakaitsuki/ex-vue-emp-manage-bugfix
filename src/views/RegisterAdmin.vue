<template>
  <div class="container">
    <div class="row register-page">
      <form class="col s12" id="reg-form">
        <div class="row">
          <p v-if="lastNameError">※姓を入力してください</p>
          <p v-if="firstNameError">※名を入力してください</p>
          <p v-if="mailAddressError">※メールアドレスを入力してください</p>
          <p v-if="passwordError">※パスワードを入力してください</p>
          <p v-if="mailDuplicateError">※登録できませんでした</p>

          <div class="input-field col s6">
            <br />
            <input
              id="last_name"
              type="text"
              class="validate"
              v-model="lastName"
              required
            />
            <label for="last_name">姓</label>
          </div>
          <div class="input-field col s6">
            <br />
            <input
              id="first_name"
              type="text"
              class="validate"
              v-model="firstName"
              required
            />
            <label for="first_name">名</label>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s12">
            <br />
            <input
              id="email"
              type="email"
              class="validate"
              v-model="mailAddress"
              required
            />
            <label for="email">メールアドレス</label>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s12">
            <br />
            <input
              id="password"
              type="password"
              class="validate"
              minlength="8"
              v-model="password"
              required
            />
            <label for="password">パスワード</label>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s6">
            <button
              class="btn btn-large btn-register waves-effect waves-light"
              type="button"
              v-on:click="registerAdmin"
            >
              登録
              <i class="material-icons right">done</i>
            </button>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import config from "@/const/const";
import axios from "axios";

/**
 * 管理者登録をする画面.
 */
@Component
export default class RegisterAdmin extends Vue {
  // 姓
  private lastName = "";
  // 名
  private firstName = "";
  // メールアドレス
  private mailAddress = "";
  // パスワード
  private password = "";
  // 姓が未入力のときのエラー
  private firstNameError = false;
  // 名が未入力のときのエラー
  private lastNameError = false;
  // メールアドレスが未入力のときのエラー
  private mailAddressError = false;
  // パスワードが未入力のときのエラー
  private passwordError = false;
  // メールアドレスが重複しているときのエラー
  private mailDuplicateError = false;

  /**
   * 管理者情報を登録する.
   *
   * @remarks
   * 本メソッドは非同期でWebAPIを呼び出し管理者登録をするためasync/await axiosを利用しています。これらを利用する場合は明示的に戻り値にPromiseオブジェクト型を指定する必要があります。
   * @returns Promiseオブジェクト
   */
  async registerAdmin(): Promise<void> {
    // 管理者登録処理

    this.lastNameError = false;
    this.firstNameError = false;
    this.mailAddressError = false;
    this.passwordError = false;

    if (this.lastName == "") {
      this.lastNameError = true;
    }
    if (this.firstName == "") {
      this.firstNameError = true;
    }
    if (this.mailAddress == "") {
      this.mailAddressError = true;
    }
    if (this.password == "") {
      this.passwordError = true;
    }
    if (
      this.lastNameError == true ||
      this.firstNameError == true ||
      this.mailAddressError == true ||
      this.passwordError == true
    ) {
      return;
    }

    const response = await axios.post(`${config.EMP_WEBAPI_URL}/insert`, {
      name: this.lastName + " " + this.firstName,
      mailAddress: this.mailAddress,
      password: this.password,
    });
    if (response.data.status == "error") {
      this.mailDuplicateError = true;
      return;
    }
    console.dir("response:" + JSON.stringify(response));
    this.$router.push("/loginAdmin");
  }
}
</script>

<style scoped>
.register-page {
  width: 600px;
}
</style>
