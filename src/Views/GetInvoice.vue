<script setup>
import { useRoute, useRouter } from "vue-router";
import { notification } from "ant-design-vue";
import { ref, onMounted, computed } from "vue";
import {
  updateInvoiceStatus,
  updateUnpaidInvoiceStatus,
  deleteInvoice,
  getSingleInvoice,
} from "../service/invoiceService";
import Header from "../components/Header.vue";
import { Colors } from "../utils/color";
import { useInvoiceStore } from "../stores/index"; 
import Swal from "sweetalert2";
import axiosInstance from "../service/axios";
const logoPreview = ref("");
const invoice = useInvoiceStore();
const router = useRouter();
const route = useRoute();
const invoiceId = route.params._id;
const dropdownTitle = "Actions";
const dropdownItems = [
  { title: "Edit" },
  { title: "Download as Pdf" },
  { title: "Mark as Paid" },
  { title: "Mark as Unpaid" },
  { title: "Send Invoice" },
  { title: "Delete" },
];

const isLoading = ref(false);
const business = ref("");
const clientId = ref("");

const changeStatus = async () => {
  try {
    isLoading.value = true;
    const updateData = {
      paymentStatus: "Paid",
    };
    const { success, data, error } = await updateInvoiceStatus(
      invoiceId,
      updateData
    );

    if (success) {
      router.push("/Index");
      openNotificationWithIcon(
        "success",
        data.message || "Payment Method has been Updated successfully."
      );
    } else {
      console.error("Error During Payment Method updation:", error);
      openNotificationWithIcon(
        "error",
        error || "error During Payment Method  Updation."
      );
      if (
        error === "Your subscription plan has expired. Please update your plan."
      ) {
        router.push("/subscription");
      } else {
        openNotificationWithIcon("error", error);
      }
    }
  } catch (error) {
    console.error("Error During Payment Method Updation:", error);
    openNotificationWithIcon(
      "error",
      "An error occurred while updating the Payment Method."
    );
  } finally {
    isLoading.value = false;
  }
};
const openNotificationWithIcon = (type, message) => {
  notification[type]({
    message: type === "success" ? "Success" : "Error",
    description: message,
    duration: 3,
  });
};
const changeUnpaidStatus = async () => {
  try {
    isLoading.value = true;
    const updateData = {
      paymentStatus: "Unpaid",
    };
    const { success, data, error } = await updateUnpaidInvoiceStatus(
      invoiceId,
      updateData
    );

    if (success) {
      router.push("/Index");
      openNotificationWithIcon(
        "success",
        data.message || "Payment Method has been Updated successfully."
      );
    } else {
      console.error("Error During Payment Method updation:", error);
      openNotificationWithIcon(
        "error",
        error || "error During Payment Method  Updation."
      );

      if (
        error === "Your subscription plan has expired. Please update your plan."
      ) {
        router.push("/subscription");
      } else {
        openNotificationWithIcon("error", error);
      }
    }
  } catch (error) {
    console.error("Error During Payment Method Updation:", error);
    openNotificationWithIcon(
      "error",
      "An error occurred while updating the Payment Method."
    );
  } finally {
    isLoading.value = false;
  }
};

const deleteInvoicee = async () => {
  try {
    const status = await deleteInvoice(invoiceId);
    router.push("/Index");
    openNotificationWithIcon(
      "success",
      data.message || "Invoice has been deleted successfully."
    );
  } catch (error) {
    console.error("Error deleting invoice Deletion:", error);
  }
};
const invoiceDetails = ref("");
const imageUrl = computed(() => {
  return invoiceDetails.value.url
    ? invoiceDetails.value.url
    : "https://res.cloudinary.com/dfbsbullu/image/upload/v1709745593/iribv5nqn6iovph3buhe.png";
});
onMounted(async () => {
  try {
    isLoading.value = true;
    const response = await getSingleInvoice(invoiceId);
    invoiceDetails.value = response.data;
    business.value = invoiceDetails.value.sender;
    clientId.value = invoiceDetails.value.receiver._id;
    invoice.updateFormData(invoiceDetails);
    logoPreview.value = invoiceDetails.logoPreview;
  } catch (error) {
    console.error("Error fetching GetInvoice details:", error);
  } finally {
    isLoading.value = false;
  }
});

const getInvoiceLink = () => {
  // return `http://3. 1.100.174:5173/invoice/${invoiceId}`;
  return `http://localhost:5173/invoice/${invoiceId}`;
};
const handleDropdownItemClickParent = (clickedItem) => {
  if (clickedItem.title === "Download as Pdf") {
    const url = new URL(axiosInstance.defaults.baseURL);
    // url.port = "3010";
    url.pathname = "/api/pdf/X-Invoice";
    url.searchParams.append("clientId", clientId.value);
    url.searchParams.append("businessId", business.value._id);
    url.searchParams.append("invoiceId", invoiceId);
    window.open(url.toString(), "_blank");
  } else if (clickedItem.title === "Edit") {
    router.push(`/GetInvoice/${invoiceId}/edit`);
  } else if (clickedItem.title === "Mark as Paid") {
    changeStatus(invoiceId);
    router.push("/Index");
  } else if (clickedItem.title === "Delete") {
    deleteInvoicee(invoiceId);
  } else if (clickedItem.title === "Send Invoice") {
    router.push(`/GetInvoice/${invoiceId}/send`);
  } else if (clickedItem.title === "Mark as Unpaid") {
    changeUnpaidStatus(invoiceId);
    router.push("/Index");
  }
};
const info = () => {
  Swal.fire({
    title: "Info",
    html: `Description Item's are Resizeable`,
    confirmButtonText: "Close",
  });
};
</script>

<template>
  <div v-if="isLoading">
    <a-space class="w-full p-8 flex justify-center mt-[25%]">
      <a-spin size="large" />
    </a-space>
  </div>

  <div v-else>
    <div class="bg-white">
      <Header
        headerTitle="View Invoice"
        backButtonText=" &nbsp &lt Invoices &nbsp  &nbsp"
        backRoute="Index"
        :dropdownTitle="dropdownTitle"
        saveDraftButtonText="Edit"
        :showDraftButton="true"
        :dropdownItems="dropdownItems"
        :saveDraftButtonColor="Colors.orange"
        :showDropdown="true"
        :onDropdownItemClick="handleDropdownItemClickParent"
      />
    </div>
<div class="bg-gray-200 h-[max-content] p-4">
    <section
      class="w-[100%] 2xl:w-[50%] md:w-[93%] lg:w-[85%] xl:w-[63%]"
    >
      <form @submit.prevent class="p-4 bg-gray-100">
        <div class="border border-black py-4 bg-gray-200">
          <div class="flex justify-center items-center">
            <p class="px-2">
              <!-- Link To Invoice: http://3.1.100.174:5173/invoice/{{ -->
              Link To Invoice: http://3.1.100.174:5173/invoice/{{
                invoiceDetails._id
              }}
              <a
                class="text-blue-700 cursor-pointer"
                :href="getInvoiceLink()"
                target="_blank"
                >(Preview)</a
              > 
            </p>
          </div>
        </div>
        <div class="grid grid-cols-2 items-center">
          <div class="flex w-full mt-8 pl-6">
            <div
              class="flex mr-5 items-center justify-center text-xl w-[60px] max-h-12 text-black"
            >
              <span
                class="px-[15px] py-[4px] rounded"
                :class="{
                  'bg-[#10C0CB] text-white text-[12px] ':
                    invoiceDetails.paymentStatus === 'Paid',
                  'bg-[#FFB74D] text-white text-[12px]':
                    invoiceDetails.paymentStatus === 'Unpaid',
                  'bg-[#bababa] text-white text-[12px]':
                    invoiceDetails.paymentStatus === 'Draft',
                }"
              >
                {{ invoiceDetails.paymentStatus }}
              </span>
            </div>
            <div class="flex mt-2">
              <p class="text-xl font-semibold">
                {{ invoiceDetails.invoiceName }}
              </p>
            </div>
          </div>

          <div
            class="w-48 mt-4 lg:ml-[25%] 2xl:ml-[30%] h-auto flex justify-end items-end"
          >
            <img :src="imageUrl" alt="Logo" />
          </div>
        </div>
        <div class="flex mb-8 mt-4 pl-4">
          <div class="flex flex-col mb-2">
            <p class="font-semibold">From:</p>
            <div class="text-left">
              <p class="">
                <span class="">
                  Selected Profile Type:

                  {{ business.profileType }} </span
                ><br />
                <span v-if="business.firstName"
                  >{{ business.firstName }}&nbsp;</span
                >
                <span v-if="business.lastName"
                  >{{ business.lastName }}<br
                /></span>
                <span v-if="business.address1"
                  >{{ business.address1 }}&nbsp;</span
                >
                <span v-if="business.address2">{{ business.address2 }}</span
                ><br />
                <span v-if="business.postalCode"
                  >{{ business.postalCode }}&nbsp;
                </span>
                <span v-if="business.city">{{ business.city }}&nbsp;</span>
                <span v-if="business.state">{{ business.state }}</span
                ><br />
                <span v-if="business.email">{{ business.email }}<br /></span>
              </p>
            </div>
            <br />
            <p class="font-semibold">To:</p>

            <div class="text-left">
              <p class="" v-if="invoiceDetails.receiver">
                <span v-if="invoiceDetails.receiver.firstName"
                  >{{ invoiceDetails.receiver.firstName }}&nbsp;</span
                >
                <span v-if="invoiceDetails.receiver.lastName"
                  >{{ invoiceDetails.receiver.lastName }}<br
                /></span>
                <span v-if="invoiceDetails.receiver.address1"
                  >{{ invoiceDetails.receiver.address1 }}&nbsp;</span
                >
                <span v-if="invoiceDetails.receiver.address2">{{
                  invoiceDetails.receiver.address2
                }}</span
                ><br />
                <span v-if="invoiceDetails.receiver.postalCode"
                  >{{ invoiceDetails.receiver.postalCode }}&nbsp;<br
                /></span>
                <span v-if="invoiceDetails.receiver.city"
                  >{{ invoiceDetails.receiver.city }}&nbsp;</span
                >
                <span v-if="invoiceDetails.receiver.state">{{
                  invoiceDetails.receiver.state
                }}</span
                ><br />
                <span v-if="invoiceDetails.receiver.email">{{
                  invoiceDetails.receiver.email
                }}</span
                ><br />
              </p>
              <p v-else>Receiver Details Unavailable</p>
            </div>
          </div>
          <div
            class="flex flex-col md:ml-[10%] lg:ml-[30%] xl:ml-[38%] 2xl:ml-[40%] mt-4"
          >
            <div>
              <p class="font-semibold">Invoice No</p>
              <div class="text-left">{{ invoiceDetails.invoiceNumber }}</div>
            </div>
            <div class="mb-4 mt-4">
              <p class="font-semibold">Date</p>
              <div class="text-left">{{ invoiceDetails.date }}</div>
            </div>
            <div class="mb-4 mt-4">
              <p class="font-semibold">Invoice Due</p>
              <div class="text-left">{{ invoiceDetails.invoiceDueDate }}</div>
            </div>
            <div class="mb-4 mt-4">
              <p class="font-semibold">Purchase order Number</p>
              <div class="text-left">
                {{ invoiceDetails.purchaseOrderNumber }}
              </div>
            </div>
          </div>
        </div>
        <br />
        <div class="border mx-6"></div>
        <br />
        <div class="flex lg:flex-row 2xl:flex-row xl:flex-row flex-col px-4">
          <table class="w-full">
            <thead>
              <tr class="bg-gray-100 h-10">
                <th
                  class="text-center font-semibold cursor-pointer"
                  @click="info"
                >
                  Description
                </th>
                <th class="font-semibold text-center">Quantity</th>
                <th class="font-semibold text-center">Rate</th>
                <th class="font-semibold text-center">Amount</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in invoiceDetails.items" :key="index">
                <td style="width: 25%">
                  <textarea
                    disabled
                    cols="25"
                    rows="2"
                    v-model="item.description"
                    class="w-full h-full pl-0 text-justify"
                    style="overflow: hidden"
                  ></textarea>
                </td>
                <td style="width: 25%">
                  <input
                    disabled
                    class="w-full h-full text-center"
                    v-model="item.quantity"
                  />
                </td>
                <td style="width: 25%">
                  <input
                    disabled
                    class="w-full h-full text-center"
                    v-model="item.rate"
                  />
                  <!-- <select disabled class="w-full h-full mt-2">
            <option v-for="unit in item.unit" :key="unit" :value="unit">
              {{ unit.name }}
            </option>
          </select> -->
                </td>
                <td style="width: 25%">
                  <input
                    disabled
                    class="w-full h-full text-center"
                    v-model="item.amount"
                  />
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <br />
        <div class="border mx-6"></div>
        <!-- <div
            style="text-align: left; margin-left: 10px"
            class="w-[50%]"
          ></div> -->

        <!-- <div class=" ">
                <span class="">Date</span>

                <input
                  readonly
                  class="w-[53vw] text-right border-0 bg-white"
                  placeholder="68970.00"
                />
              </div>
              <hr /> -->
        <div class="flex flex-col max-w-full items-end xl:mr-20">
          <div
            class="flex justify-between xl:w-[70%] md:w-[75%] 2xl:w-[70%] items-end"
          >
            <div class="text-black flex">
              <span
                class="px-[4px] py-[10px] font-semibold border-black text-[12px] rounded"
              >
                SubTotal
              </span>
            </div>
            <div class="pb-2 pt-4 mr-8">
              {{ invoiceDetails.subtotal }} {{ invoiceDetails.currency }}
            </div>
            <!-- <div class="text-black">
              <span class=""> Total </span>
            </div> -->

            <!-- :value="getSubtotal()" -->
          </div>

          <div
            class="flex justify-between xl:w-[70%] md:w-[75%] 2xl:w-[70%] items-end"
          >
            <div class="">
              <span
                class="px-[6px] font-semibold py-[10px] border-black text-[12px] rounded"
                :class="{
                  'bg-[#10C0CB] text-white  ':
                    invoiceDetails.paymentStatus === 'Paid',
                  'bg-[#FFB74D] text-white':
                    invoiceDetails.paymentStatus === 'Unpaid',
                  'bg-[#bababa] text-white':
                    invoiceDetails.paymentStatus === 'Draft',
                }"
              >
                Balance
              </span>
            </div>
            <div class="pt-2 mr-8">
              {{ invoiceDetails.subtotal }} {{ invoiceDetails.currency }}
            </div>
            <!-- lg:w-[53vw] 2xl:w-[53vw]  md:w-32 -->
          </div>
        </div>

        <br />

        <div class="container flex pl-6">
          <div class="flex-left">
            <div class="text-left mt-8">
              <div>
                <div class="flex w-full">
                  <p class="font-semibold">Invoice Notes :</p>
                </div>
                <!-- v-model="invoiceDetails.notes" -->
                <div>{{ invoiceDetails.notes }}</div>
              </div>
            </div>
          </div>
        </div>
        <div class="border mx-6 my-8"></div>
        <div class="flex pl-6">
          <div class="text-left font-semibold">Email :</div>
          <div class="text-left mx-2">
            {{ business.email }}
          </div>
        </div>
      </form>
    </section>
  </div>
  </div>
</template>
<style scoped>
p {
  text-align: left;
}

input,
textarea,
select {
  border: none !important;
  background: none;
  border-radius: 0;
  outline: none;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
}
.resizable-textarea {
  resize: both;
  overflow: auto;
}
</style>
