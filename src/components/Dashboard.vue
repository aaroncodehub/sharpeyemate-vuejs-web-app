<template>
  <v-container grid-list-xs v-if="loading">
    <v-row class="fill-height" align-content="center" justify="center">
      <v-col class="subtitle-1 text-center" cols="12">Getting your orders</v-col>
      <v-col cols="6">
        <v-progress-linear color="primary" indeterminate rounded height="6"></v-progress-linear>
      </v-col>
    </v-row>
  </v-container>
  <v-container fluid grid-list-md v-else>
    <!-- <myAlert v-if="error" @dismissed="onDismissed" :text="error"></myAlert> -->
    <v-alert
      v-if="error"
      border="top"
      colored-border
      type="error"
      elevation="2"
    >{{ error }} ! Oops. something went wrong , please try again later.</v-alert>

    <v-data-table :headers="headers" :items="orders" :search="search" class="elevation-1">
      <template v-slot:top>
        <v-toolbar flat color="white">
          <v-toolbar-title>
            Sales Orders
            <!-- <v-btn :to="{name: 'CreateOrder'}" fab bottom  color="red darken-3" dark fixed>
              <v-icon>add</v-icon>
            </v-btn>-->
          </v-toolbar-title>
          <div class="flex-grow-1"></div>
          <v-text-field
            v-model="search"
            append-icon="search"
            label="Search your order"
            single-line
            hide-details
          ></v-text-field>
        </v-toolbar>
      </template>

      <template v-slot:body="{ items }">
        <tbody>
          <tr v-for="item in items" :key="item.name">
            <td :class="`pa-2 order ${item.sales_order_status}`">
              <v-tooltip bottom>
                <template v-slot:activator="{ on }">
                  <span v-on="on">{{ item.sales_order_status }}</span>
                </template>
                <span>draft : It has not been confirmed yet</span>
                <br />
                <span>confirmed : confirmed by sharpeye</span>
              </v-tooltip>
            </td>
            <td>
              <v-tooltip bottom>
                <template v-slot:activator="{ on }">
                  <span v-on="on">
                    {{
                    item.manufacturing_status_of_sales_order
                    }}
                  </span>
                </template>
                <span>ongoing : It is still in progress</span>
                <br />
                <span>done : The glass is ready to go</span>
              </v-tooltip>
            </td>
            <td>{{ item.name }}</td>
            <td>{{ item.client_order_ref | lessLetters }}</td>
            <td>{{ item.customer_request | lessLetters }}</td>
            <td>
              <!-- <v-btn small text @click="download(item)">
                <v-icon color="primary" class="mr-2">save_alt</v-icon>
              </v-btn>-->
              <!-- <img src="data:image/png;base64,R0lGODlhDAAMAKIFAF5LAP/zxAAAANyuAP/gaP///wAAAAAAACH5BAEAAAUALAAAAAAMAAwAAAMlWLPcGjDKFYi9lxKBOaGcF35DhWHamZUW0K4mAbiwWtuf0uxFAgA7"> -->
              <!-- <a href="data:application/pdf;base64,R0lGODlhDAAMAKIFAF5LAP/zxAAAANyuAP/gaP///wAAAAAAACH5BAEAAAUALAAAAAAMAAwAAAMlWLPcGjDKFYi9lxKBOaGcF35DhWHamZUW0K4mAbiwWtuf0uxFAgA7" download="test.png">test</a>
              -->
              <!-- <v-select
                v-model="e1"
                :items="states"
                menu-props="auto"
                label="Select Drawings"
                hide-details
                prepend-icon="map"
                single-line
                @click="download(item)"
              ></v-select>-->
              <v-menu offset-y rounded>
                <template v-slot:activator="{ on, attrs }">
                  <v-btn color="primary" v-bind="attrs" v-on="on" @click="download(item)">
                    <v-icon>save_alt</v-icon>
                  </v-btn>
                </template>
                <v-list>
                  <v-list-item v-if="dLoading">
                    <v-btn
                      :loading="dLoading"
                      :disabled="dLoading"
                      tile large icon
                    >
                    </v-btn>
                  </v-list-item>
                  <v-list-item
                    v-for="(item, index) in myAttachments"
                    :key="index"
                    @click="toDownload(item)"
                  >
                    <v-list-item-title>{{ item.name }}</v-list-item-title>
                  </v-list-item>
                </v-list>
              </v-menu>
            </td>

            <td>
              <v-tooltip top v-if="item.state !== 'cancel' && item.sales_order_status == 'draft'">
                <template v-slot:activator="{ on }">
                  <v-btn small text @click="openUploadDialog(item)" v-on="on" icon>
                    <v-icon color="primary" class="mr-2">cloud_upload</v-icon>
                  </v-btn>
                </template>
                <span>Upload Your Drawing</span>
              </v-tooltip>
              <v-tooltip top v-else>
                <template v-slot:activator="{ on }">
                  <v-btn small text v-on="on" icon>
                    <v-icon disabled color="primary" class="mr-2">cloud_upload</v-icon>
                  </v-btn>
                </template>
                <span>
                  You have to contact us once the order is confirmed by
                  Sharpeye
                </span>
              </v-tooltip>

              <v-tooltip top>
                <template v-slot:activator="{ on }">
                  <v-btn small text @click="openMoreDialog(item)" v-on="on" icon>
                    <v-icon color="primary" class="mr-2">remove_red_eye</v-icon>
                  </v-btn>
                </template>
                <span>Review Your Order</span>
              </v-tooltip>

              <v-tooltip top v-if="item.state !== 'cancel' && item.sales_order_status == 'draft'">
                <template v-slot:activator="{ on }">
                  <v-btn small text @click="editOrder(item)" v-on="on" icon>
                    <v-icon color="primary" class="mr-2">edit</v-icon>
                  </v-btn>
                </template>
                <span>Edit Your Order</span>
              </v-tooltip>
              <v-tooltip top v-else>
                <template v-slot:activator="{ on }">
                  <v-btn small text v-on="on" icon>
                    <v-icon disabled color="primary" class="mr-2">edit</v-icon>
                  </v-btn>
                </template>
                <span>
                  You have to contact us once the order is confirmed by
                  Sharpeye
                </span>
              </v-tooltip>

              <v-tooltip top v-if="item.state !== 'cancel' && item.sales_order_status == 'draft'">
                <template v-slot:activator="{ on }">
                  <v-btn small text @click="openDeleteDialog(item)" v-on="on" icon>
                    <v-icon color="primary" class="mr-2">delete_forever</v-icon>
                  </v-btn>
                </template>
                <span>Delete Your Order</span>
              </v-tooltip>
              <v-tooltip top v-else>
                <template v-slot:activator="{ on }">
                  <v-btn small text v-on="on" icon>
                    <v-icon disabled color="primary" class="mr-2">delete_forever</v-icon>
                  </v-btn>
                </template>
                <span>
                  You have to contact us once the order is confirmed by
                  Sharpeye
                </span>
              </v-tooltip>
            </td>
          </tr>
        </tbody>
      </template>
    </v-data-table>

    <!-- Delete Dialog -->
    <div class="text-center">
      <v-dialog v-model="dialog.deleteDialog" width="500">
        <v-card>
          <v-card-title class="headline grey lighten-2" primary-title>Warning Message</v-card-title>
          <div>
            <p class="my-3 mx-3 red--text">Are you sure you want to delete {{ order.name }} ?</p>
          </div>
          <v-divider></v-divider>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" @click="deleteOrder">ACCEPT</v-btn>
            <v-btn color="red darken-3" text @click="dialog.deleteDialog = false">CANCEL</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </div>

    <!-- More Dialog -->
    <v-dialog
      v-model="dialog.moreDialog"
      persistent
      :overlay="false"
      max-width="1200px"
      transition="dialog-transition"
    >
      <v-stepper v-model="stepMore" vertical>
        <v-stepper-step :complete="stepMore > 1" step="1">
          Sales order details -- {{ order.name }}
          <small>sales order information</small>
        </v-stepper-step>

        <v-stepper-content step="1">
          <v-card flat color="grey lighten-5" class="mb-6 px-2">
            <v-layout wrap>
              <v-flex xs12 md6>
                <v-text-field
                  prepend-icon="location_on"
                  label="Shipping Address"
                  v-model.trim="order.alternate_shipping_address"
                ></v-text-field>
              </v-flex>
              <v-flex xs12 md6>
                <v-text-field
                  prepend-icon="description"
                  label="Reference/Description"
                  v-model.trim="order.client_order_ref"
                ></v-text-field>
              </v-flex>
              <v-flex xs12 md4>
                <v-text-field
                  prepend-icon="contacts"
                  label="Contact Name"
                  v-model.trim="order.contact_name"
                ></v-text-field>
              </v-flex>
              <v-flex xs12 md4>
                <v-text-field
                  prepend-icon="phone"
                  label="Contact Phone"
                  v-model.trim="order.mobile"
                ></v-text-field>
              </v-flex>
              <v-flex xs12 md4>
                <v-select
                  prepend-icon="local_shipping"
                  v-model="order.delivery_method"
                  :items="deliver"
                  label="Deliver Method"
                ></v-select>
              </v-flex>
              <v-flex v-if="order.customer_request">
                <v-textarea
                  prepend-icon="info"
                  label="Special Request"
                  v-model.trim="order.customer_request"
                ></v-textarea>
              </v-flex>
            </v-layout>
            <v-layout wrap v-for="(orderLine, index) in order.order_lines" v-bind:key="index">
              <v-layout
                wrap
                v-if="
                  orderLine.product_id[0] == 2380 ||
                    orderLine.product_id[0] == 22 ||
                    orderLine.product_id[0] == 23 ||
                    orderLine.product_id[0] == 24 ||
                    orderLine.product_id[0] == 25 ||
                    orderLine.product_id[0] == 2522 ||
                    orderLine.product_id[0] == 2303 ||
                    orderLine.product_id[0] == 26 ||
                    orderLine.product_id[0] == 27 ||
                    orderLine.product_id[0] == 28 ||
                    orderLine.product_id[0] == 29 ||
                    orderLine.product_id[0] == 2083 ||
                    orderLine.product_id[0] == 2529 ||
                    orderLine.product_id[0] == 30 ||
                    orderLine.product_id[0] == 2530
                "
              >
                <v-flex xs12 md4>
                  <v-select
                    v-model="orderLine.product_id[0]"
                    :items="products"
                    item-text="product"
                    item-value="value"
                    label="Product"
                  ></v-select>
                </v-flex>
                <v-flex xs12 md2>
                  <v-text-field v-model.number="orderLine.width" label="Width"></v-text-field>
                </v-flex>
                <v-flex xs12 md2>
                  <v-text-field v-model.number="orderLine.height" label="Height"></v-text-field>
                </v-flex>
                <v-flex xs12 md2>
                  <v-text-field v-model.number="orderLine.number_of_pieces" label="PCs"></v-text-field>
                </v-flex>
                <v-flex xs12 md2>
                  <v-select
                    v-model="orderLine.ynt_selection"
                    :items="drawings"
                    label="Flag"
                    item-text="flag"
                    item-value="value"
                  ></v-select>
                </v-flex>
              </v-layout>
            </v-layout>
          </v-card>
          <v-btn color="primary" @click="dialog.moreDialog = false">close</v-btn>
        </v-stepper-content>
      </v-stepper>
    </v-dialog>

    <!-- upload dialog -->

    <v-dialog
      v-model="dialog.uploadDialog"
      persistent
      :overlay="false"
      max-width="1200px"
      transition="dialog-transition"
    >
      <v-stepper v-model="step" vertical>
        <v-stepper-step :complete="step > 1" step="1">
          Upload the drawing & attachment -- {{ order.name }}
          <small>drawing & attachment</small>
        </v-stepper-step>

        <v-stepper-content step="1">
          <v-card flat class="mb-6 px-2">
            <v-layout wrap>
              <v-flex xs12>
                <v-alert
                  v-if="message"
                  border="top"
                  colored-border
                  type="error"
                  elevation="2"
                >{{ message }}</v-alert>
                <v-file-input
                  :rules="rules"
                  placeholder="Select the drawing or attachment"
                  small-chips
                  label="Drawing"
                  type="file"
                  ref="drawing"
                  @change="fileEncode"
                ></v-file-input>
              </v-flex>
            </v-layout>
          </v-card>
          <v-btn
            :disabled="loading"
            :loading="loading"
            color="primary"
            @click="uploadFile(order.id, order.name)"
          >
            Upload Drawing
            <v-icon right dark>cloud_upload</v-icon>
          </v-btn>
          <v-btn text @click="dialog.uploadDialog = false">close</v-btn>
        </v-stepper-content>
      </v-stepper>
    </v-dialog>

    <!-- Agreement -->
    <v-dialog
      v-model="userProfile.agreement"
      persistent
      :overlay="false"
      max-width="1000px"
      transition="dialog-transition"
    >
      <v-stepper v-model="stepMore" vertical>
        <v-stepper-step step="1">
          SharpeyeMate Agreement
          <small>Terms ans Conditions</small>
        </v-stepper-step>

        <v-stepper-content step="1">
          <v-card flat color="grey lighten-5" class="mb-6 px-2">
            <v-layout wrap>
              <v-flex xs12>
                <p>
                  Dear customer,
                  <br />
                  <br />When you first use Sharpeye Mate you will need to log in with your provided username and you have to reset your password to keep your account secure. Log in information is confidential and Sharpeye Ltd do not have access to your login details.
                  Please keep your log in information secure to ensure that others are unable to access your Sharpeye Mate account.
                  <br />
                  <br />If your account is used by another person to place glass orders, you will be personally liable to pay for those orders.
                  <br />
                  <br />Sharpeye Mate is your main method to place your glass orders. Sharpeye Ltd will regard all glass orders placed from your account as valid. If you need to cancel or change any details of your confirmed glass order please contact us via phone or email.
                  <br />
                  <br />Sharpeye Ltd reserve the right to amend these terms and conditions, in whole or in part, for any reason at any time.
                  <br />
                  <br />The Sharpeye Mate App is governed by, and will be construed in accordance with the New Zealand laws and the courts of New Zealand will have non-exclusive jurisdiction.
                  <br />
                </p>
                <p class="font-weight-light">
                  Please note that the notification will be mailed to
                  {{ userProfile.noticeEmail }} once the new sales order is
                  created.
                </p>
              </v-flex>
            </v-layout>
          </v-card>
          <v-btn :disabled="loading" :loading="loading" color="primary" @click="agree">
            Agree
            <v-icon right dark>mdi-checkbox-marked-circle</v-icon>
          </v-btn>
          <v-btn text color="primary" @click="disagree">Disagree</v-btn>
        </v-stepper-content>
      </v-stepper>
    </v-dialog>

    <v-btn bottom color="red darken-3" dark fab fixed right :to="{ name: 'CreateOrder' }">
      <v-icon>mdi-plus</v-icon>
    </v-btn>
    <v-snackbar color="success" v-model="snackbar.snackbar" :timeout="snackbar.timeout">
      {{ snackbar.text }}
      <v-btn text @click="snackbar.snackbar = false">Close</v-btn>
    </v-snackbar>
  </v-container>
</template>

<script>
import axios from "axios";
const fb = require("../fb");
import { mapState } from "vuex";
export default {
  data() {
    return {
      dLoading: false,
      myAttachments: [],
      myItems: [
        { title: "Click Me" },
        { title: "Click Me" },
        { title: "Click Me" },
        { title: "Click Me 2" },
      ],
      headers: [
        { text: "Sales Order Status", value: "sales_order_status" },
        {
          text: "Manufacturing Status",
          value: "manufacturing_status_of_sales_order",
        },
        {
          text: "Sales Oder Number",
          align: "left",
          value: "name",
        },
        { text: "Reference/Description", value: "client_order_ref" },
        { text: "Special Request", value: "customer_request" },
        { text: "Download Drawings", value: "" },
        { text: "Actions", value: "action", sortable: false },
      ],
      search: "",
      snackbar: {
        snackbar: false,
        text: null,
        timeout: 2000,
      },
      myAction: false,
      show: true,
      dialog: {
        deleteDialog: false,
        moreDialog: false,
        editDialog: false,
        uploadDialog: false,
      },
      order: {
        id: null,
        name: null,
        alternate_shipping_address: null,
        client_order_ref: null,
        contact_name: null,
        mobile: null,
        delivery_method: null,
        order_lines: null,
        customer_request: null,
      },
      step: 1,
      stepMore: 1,
      products: [
        { product: "LMG-4MM", value: 2380 },
        { product: "LMG-5MM", value: 22 },
        { product: "LMG-5MM-LOW-IRON", value: 23 },
        { product: "LMG-5MM-SILVER-MIRRO", value: 24 },
        { product: "LMG-6MM", value: 25 },
        { product: "LMG-6MM-SILVER-MIRRO", value: 2522 },
        { product: "LMG-6.38MM-LAMINATED", value: 2303 },
        { product: "LMG-8MM", value: 26 },
        { product: "LMG-10MM", value: 27 },
        { product: "LMG-12MM", value: 28 },
        { product: "LMG-12MM-EUROGREY", value: 29 },
        { product: "LMG-12MM-GREEN", value: 2083 },
        { product: "LMG-12MM-ULTRA-CLEAR", value: 2529 },
        { product: "LMG-15MM", value: 30 },
        { product: "LMG-15MM-ULTRA-CLEAR", value: 2530 },
      ],
      drawings: [
        { flag: "Drawing", value: "y" },
        { flag: "Dimension", value: "n" },
        { flag: "Template", value: "t" },
      ],
      deliver: ["pick_up", "deliver", "freight"],
      rules: [
        (value) =>
          !value ||
          value.size < 20000000 ||
          "The attachment size should be less than 20 MB!",
      ],
      message: null,
      base64: null,
      file: null,
    };
  },
  methods: {
    // sortBy(prop) {
    //   this.orders.sort((a, b) => (a[prop] > b[prop] ? -1 : 1));
    // },
    getColor(state) {
      if (state == "done") return "#c0392b";
      else if (state == "draft") return "#009688";
      else return "#0105ff";
    },
    deleteOrder() {
      axios({
        method: "PUT",
        url:
          "http://api-test.sharpeye.co.nz/api/v1/model/sale.order/" +
          this.order.id,
        headers: {
          access_token: this.userProfile.accessToken,
        },
        data:{
          api_cancelled: true
        }
      })
        .then(() => {
          this.$store.dispatch("deleteOrder", this.order.id);
          this.dialog.deleteDialog = false;
          this.snackbar.snackbar = true;
          this.snackbar.text = `the sales order ${this.order.name} has been deleted successfully`;
        })
        .catch((err) => {
          this.$store.commit("setError", err.message);
          // console.log(err);
        });
    },
    openDeleteDialog(order) {
      this.order.id = order.id;
      this.order.name = order.name;
      this.dialog.deleteDialog = true;
    },
    openMoreDialog(order) {
      if (order.client_order_ref) {
        this.order.client_order_ref = order.client_order_ref;
      } else {
        this.order.client_order_ref = "";
      }
      this.order.id = order.id;
      this.order.name = order.name;
      this.dialog.moreDialog = true;
      if (order.alternate_shipping_address) {
        this.order.alternate_shipping_address =
          order.alternate_shipping_address;
      } else {
        this.order.alternate_shipping_address = "";
      }
      if (order.contact_name) {
        this.order.contact_name = order.contact_name;
      } else {
        this.order.contact_name = "";
      }
      if (order.mobile) {
        this.order.mobile = order.mobile;
      } else {
        this.order.mobile = "";
      }
      this.order.customer_request = order.customer_request;
      this.order.delivery_method = order.delivery_method;
      this.order.order_lines = order.order_line;
    },
    editOrder(order) {
      this.$router.push({ name: "EditOrder", params: { order } });
    },
    openUploadDialog(order) {
      this.order.id = order.id;
      this.order.name = order.name;
      this.dialog.uploadDialog = true;
    },
    deleteOrderLine(id) {
      this.$store.commit("setLoading", true);
      axios({
        method: "DELETE",
        url:
          "http://api-test.sharpeye.co.nz/api/v1/model/sale.order.line/" + id,
        headers: {
          access_token: this.userProfile.accessToken,
        },
      })
        .then(() => {
          this.$store.commit("setLoading", false);
          this.$store.commit("deleteOrderLine", {
            lineId: id,
            orderId: this.order.id,
          });
          this.snackbar.text = " delete sales order line successfully ";
          this.snackbar.snackbar = true;
        })
        .catch((err) => {
          this.$store.commit("setLoading", false);
          this.$store.commit("setError", err.message);
          // console.log(err);
        });
    },
    fileEncode(file) {
      this.message = null;
      this.file = file;
      if (this.file && this.file.size < 20000000) {
        return new Promise((resolve, reject) => {
          const reader = new FileReader();
          reader.readAsDataURL(this.file);
          reader.onload = () => {
            let encoded = reader.result.replace(/^data:(.*,)?/, "");
            if (encoded.length % 4 > 0) {
              encoded += "=".repeat(4 - (encoded.length % 4));
            }
            resolve(encoded);
            this.base64 = encoded;
          };
          reader.onerror = (error) => reject(error);
        });
      } else {
        this.message = this.file
          ? "The attachment is too large . the max size is 20M"
          : "";
      }
    },
    uploadFile(orderId, orderName) {
      if (this.base64) {
        let formData = {
          data: this.base64,
          name: orderName + " " + "Drawing" + " " + this.file.name,
        };
        this.$store.commit("setLoading", true);
        axios({
          method: "POST",
          url:
            "http://api-test.sharpeye.co.nz/api/v1/model/sale.order/" +
            orderId +
            "/attachment",
          headers: {
            access_token: this.userProfile.accessToken,
            Accept: "application/json",
            "Content-Type": "application/json",
          },
          data: {
            attachments: [formData],
          },
        })
          .then(() => {
            this.$store.commit("setLoading", false);
            this.dialog.uploadDialog = false;
            this.snackbar.text = " upload successfully ";
            this.snackbar.snackbar = true;
          })
          .catch((err) => {
            this.$store.commit("setLoading", false);
            this.$store.commit("setError", err.message);
            // console.log(err);
          });
      }
    },
    download(item) {
      this.dLoading = true;
      this.myAttachments = [];
      axios({
        method: "GET",
        url:
          "http://api-test.sharpeye.co.nz/api/v1/model/sale.order/" +
          item.id +
          "/attachments?all_attachments=True",
        headers: {
          access_token: this.userProfile.accessToken,
          Accept: "application/json",
          "Content-Type": "application/json",
        },
      })
        .then((response) => {
          this.dLoading = false;
          if (response.data.attachments.length !== 0) {
            this.myAttachments = response.data.attachments;
          } else {
            this.myAttachments = [{ name: "no drawing" }];
          }
        })
        .catch(() => {
          this.dLoading = false;
          this.snackbar.snackbar = true;
          this.snackbar.text = `Oops ! no drawing exists`;
        });
    },
    toDownload(item) {
      const linkSource = `data:application/pdf;base64,${item.base64}`;
      const downloadLink = document.createElement("a");
      const fileName = item.name;
      downloadLink.href = linkSource;
      downloadLink.download = fileName;
      downloadLink.click();
      this.myAttachments = [];
    },
    agree() {
      this.$store.commit("setLoading", true);
      fb.usersCollection
        .doc(this.currentUser.uid)
        .update({
          agreement: false,
        })
        .then(() => {
          this.$store.commit("setLoading", false);
          this.userProfile.agreement = false;
        })
        .catch((err) => {
          this.$store.commit("setLoading", false);
          this.$store.commit("setError", err.message);
        });
    },
    disagree() {
      fb.auth
        .signOut()
        .then(() => {
          this.$store.dispatch("clearData");
          this.$router.push("/login");
        })
        .catch((err) => {
          this.$store.commit("setError", err.message);
        });
    },
  },
  computed: {
    ...mapState(["userProfile", "orders", "error", "loading", "currentUser"]),
    //   checkedOrders() {
    //     const newOrders = this.orders.map(el => {
    //       if (el.order_line.findIndex(line => line.state == "confirmed") == -1) {
    //         el.state = "draft";
    //       } else {
    //         el.state = "confirmed";
    //       }
    //       return el;
    //     });
    //     return newOrders;
    //   }
  },
  filters: {
    lessLetters: function (value) {
      if (value.length > 20) {
        value = value.slice(0, 20) + ` ...`;
        return value;
      } else if (value) {
        return value;
      }
      return "";
    },
  },
  mounted() {
    this.$store.dispatch("fetchUserProfile");
  },
  beforeDestroy() {
    this.$store.commit("clearError");
  },
};
</script>

<style>
.order.draft {
  border-left: 4px solid #009688;
}
.order.confirmed {
  border-left: 4px solid #c0392b;
}
.order.done {
  border-left: 4px solid #f83e70;
}
.order.cancel {
  border-left: 4px solid #a901f7;
}
.order.progress {
  border-left: 4px solid #ff9101;
}
.order.manual {
  border-left: 4px solid #0105ff;
}
</style>
