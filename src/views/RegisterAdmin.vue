<template>
  <div class="container">
    <div class="row register-page">
      <form class="col s12" id="reg-form">
        <div class="row">
          <p v-if="lastNameError">※姓を入力してください</p>
          <p v-if="firstNameError">※名を入力してください</p>
          <p v-if="mailAddressError">※メールアドレスを入力してください</p>
          <p v-if="passwordError">※パスワードを入力してください</p>
          <p v-if="checkPasswordError">
            ※パスワードと確認用パスワードが異なります
          </p>
          <p v-if="addressError">*住所を入力してください</p>

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
          <p v-if="zipCodeError">この郵便番号は存在しません</p>
          <div class="input-field col s10">
            <br />
            <input
              id="zipCode"
              type="text"
              class="validate"
              v-model="zipCode"
              required
            />
            <label for="first_name">郵便番号</label>
            <button type="button" v-on:click="getAddress()">住所検索</button>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s12">
            <br />
            <input
              id="address"
              type="text"
              class="validate"
              v-model="address"
              required
            />
            <label for="first_name">住所</label>
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
          <div class="input-field col s12">
            <br />
            <input
              id="checkpassword"
              type="password"
              class="validate"
              minlength="8"
              v-model="checkPassword"
              required
            />
            <label for="password">確認用パスワード</label>
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
  // 郵便番号
  private zipCode = "";
  // 住所
  private address = "";
  // メールアドレス
  private mailAddress = "";
  // パスワード
  private password = "";
  // 確認用パスワード
  private checkPassword = "";
  // 姓が未入力のときのエラー
  private firstNameError = false;
  // 名が未入力のときのエラー
  private lastNameError = false;
  // メールアドレスが未入力のときのエラー
  private mailAddressError = false;
  // パスワードが未入力のときのエラー
  private passwordError = false;
  // パスワードと確認用パスワードが異なります
  private checkPasswordError = false;
  // メールアドレスが重複しているときのエラー
  private mailDuplicateError = false;
  //存在しない郵便番号が入力されたときのエラー
  private zipCodeError = false;
  // 住所が未入力のときのエラー
  private addressError = false;

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
    this.checkPasswordError = false;
    this.addressError = false;

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
    if (this.password != this.checkPassword) {
      this.checkPasswordError = true;
    }
    if (this.address == "") {
      this.addressError = true;
    }
    if (
      this.lastNameError == true ||
      this.firstNameError == true ||
      this.mailAddressError == true ||
      this.passwordError == true ||
      this.checkPasswordError == true ||
      this.addressError == true
    ) {
      return;
    }
    //入力された内容を外部APIに送る
    const response = await axios.post(`${config.EMP_WEBAPI_URL}/insert`, {
      name: this.lastName + " " + this.firstName,
      mailAddress: this.mailAddress,
      address: this.address,
      password: this.password,
    });
    if (response.data.status == "error") {
      this.mailDuplicateError = true;
      return;
    }
    console.dir("response:" + JSON.stringify(response));
    this.$router.push("/loginAdmin");
  }
  /**
   * 郵便番号から住所を取得する
   */
  async getAddress(): Promise<void> {
    try {
      this.zipCodeError = false;
      // eslint-disable-next-line @typescript-eslint/no-var-requires
      const axiosJsonpAdapter = require("axios-jsonp");

      const response = await axios.get("https://zipcoda.net/api", {
        adapter: axiosJsonpAdapter,
        params: {
          zipcode: this.zipCode,
        },
      });
      console.dir(response);

      this.address =
        response.data.items[0].pref + response.data.items[0].address;
      // 全件返ってきたときにエラー表示にする
      if (response.data.items.length != 1) {
        this.zipCodeError = true;
        this.address = "";
      }
    } catch (error) {
      this.zipCodeError = true;
      this.address = "";
    }
  }
}
</script>

<style scoped>
.register-page {
  width: 600px;
}
</style>
