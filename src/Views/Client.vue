<script setup>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import { clientApi } from "../service/ClientService";
import { Colors } from "../utils/color";
import Header from "../components/Header.vue";
import { useInvoiceStore } from "../stores/index";
import { notification } from "ant-design-vue";
import { uploadImage } from "../service/UploadImage";
const isLoader = ref(false);
const router = useRouter();
const isLoading = ref(false);
const isLoadingImg = ref(false);
const submitclientDataOrganization = async () => {
  if (isLoadingImg.value) {
    openNotificationWithIcon("error", "Please Wait To upload Image First");
    return;
  }

  try {
    isLoader.value = true;
    if (!validateForm()) return;
    const clientData = {
      ...invoice.userClientProfile.clientDataOrganization,
      clientType: "organization",
    };
    const { success, data, error } = await clientApi(clientData);

    if (success) {
      openNotificationWithIcon("success", data.message || "Client has been Created successfully.")
      return true;
    } else {
      console.error("Error During Client organization:", error);
      openNotificationWithIcon("error", error || "An error occurred while creating the client.")
      if (
        error === "Your subscription plan has expired. Please update your plan."
      ) {
        router.push("/subscription");
      } else {
        openNotificationWithIcon("error", error);
      } return false;
    }
  } catch (error) {
    console.error("Error During Client organization:", error);
    openNotificationWithIcon(
      "error",
      "An error occurred while creating the client."
    );
  } finally {
    isLoader.value = false;
  }
};
const submitclietDataindividual = async () => {
  if (isLoadingImg.value) {
    openNotificationWithIcon("error", "Please Wait To upload Image First");
    return;
  }

  try {
    isLoader.value = true;
    if (!validateForm()) return;
    const clientData = {
      ...invoice.userClientProfile.clientDataindividual,
      clientType: "individual",
    };
    const { success, data, error } = await clientApi(clientData);

    if (success) {
      openNotificationWithIcon("success", data.message || "Client has been Created successfully.")
      return true;
    } else {
      console.error("Error During Client individual:", error);
      openNotificationWithIcon("error", error || "An error occurred while creating the client.")
      if (error === "Maximum clients limit reached for the user") {
        router.push("/subscription");
      } else {
        openNotificationWithIcon("error", error);
      }return false;
    }
    
  } catch (error) {
    console.error("Error During Client individual:", error);
    openNotificationWithIcon(
      "error",
      "An error occurred while creating the client."
    );
  } finally {
    isLoader.value = false;
  }
};
const invoice = useInvoiceStore();
const validateForm = () => {
  const emptyFields = [];

  if (!getProfileData().firstName) {
    emptyFields.push("FirstName ");
  }
  if (!getProfileData().lastName) {
    emptyFields.push("LastName ");
  }
  if (!getProfileData().phone) {
    emptyFields.push("Phone Number");
  }
  if (!getProfileData().email) {
    emptyFields.push(" Email must contain '@'");
  } else if (
    !getProfileData().email.includes("@") ||
    !/.+\@.+\..+/.test(getProfileData().email)
  ) {
    emptyFields.push("Email must be valid and contain '@'.");
  }
  if (!getProfileData().state) {
    emptyFields.push(" State ");
  }
  if (!getProfileData().city) {
    emptyFields.push(" City ");
  }
  if (!getProfileData().country) {
    emptyFields.push("Country ");
  }

  if (emptyFields.length > 0) {
    const alertMessage = `Please fill in the following required fields: and ${emptyFields.join(
      ", "
    )}`;
    openNotificationWithIcon("error", alertMessage);
    return false;
  }

  return true;
};
const selectedField = ref("individual");
const selectField = (field) => {
  selectedField.value = field;
};
const openNotificationWithIcon = (type, message) => {
  notification[type]({
    message: type === "success" ? "Success" : "Error",
    description: message,
    duration: 5,
  });
};
const handleSaveDraftButtonClick = async () => {
  try {
    if (selectedField.value === "individual") {
      const success = await submitclietDataindividual();
      if(success){
        router.push("/AllClients")
      }
    } else if (selectedField.value === "organization") {
      const success = await submitclientDataOrganization();
      if(success){
        router.push("/AllClients")
      }
    }
  } catch (error) {
    console.error("Error during form submission:", error);
  }
};const handleSaveDraftButton = async () => {
  try {
    if (selectedField.value === "individual") {
      const success = await submitclietDataindividual();
      if(success){
        router.push("/new")
      }
    } else if (selectedField.value === "organization") {
      const success = await submitclientDataOrganization();
      if(success){
        router.push("/new")
      }
    }
  } catch (error) {
    console.error("Error during form submission:", error);
  }
};

const logoInputRef = ref(null);
const logoPreview = ref(null);
const handleFileInputChange = async () => {
  const file = logoInputRef.value.files[0];
  if (file) {
    try {
      isLoadingImg.value = true;
      const imageUrl = await uploadImage(file);
      if (imageUrl) {
        if (selectedField.value === "individual") {
          invoice.userClientProfile.clientDataindividual.url = imageUrl;
        } else if (selectedField.value === "organization") {
          invoice.userClientProfile.clientDataOrganization.url = imageUrl;
        }
        displayImage(logoInputRef.value, imageUrl);
      } else {
        console.error("Failed to upload file");
      }
    } catch (error) {
      console.error("Error uploading file:", error);
    } finally {
      isLoadingImg.value = false;
    }
  }
};

const displayImage = (input, imageUrl) => {
  const file = input.files[0];

  if (file) {
    const reader = new FileReader();

    reader.onload = (e) => {
      if (selectedField.value === "individual") {
        invoice.userClientProfile.clientDataindividual.url = imageUrl;
      } else if (selectedField.value === "organization") {
        invoice.userClientProfile.clientDataOrganization.url = imageUrl;
      }
      const imagePreview = document.getElementById("imagePreview");
      imagePreview.src = imageUrl;
    };

    reader.readAsDataURL(file);
    if (selectedField.value === "individual") {
      invoice.userClientProfile.clientDataindividual.url = file;
    } else if (selectedField.value === "organization") {
      invoice.userClientProfile.clientDataOrganization.url = file;
    }
  }
};
onMounted(() => {
  invoice.resetState();
});
const showOptions = ref(false);
const filterOptions = (event) => {
  const input = event.target.value.toLowerCase();
  filteredOptions.value = invoice.countryOptions.filter(
    (option) => option.label.toLowerCase().indexOf(input) > -1
  );
};
const filteredOptions = ref(invoice.countryOptions);
const selectOption = (option, event) => {
  event.stopPropagation(); 
  if (selectedField.value === "individual") {
    invoice.userClientProfile.clientDataindividual.country = option.label; 
  } else if (selectedField.value === "organization") {
    invoice.userClientProfile.clientDataOrganization.country = option.label;
  }
  hideDropdown(); 
};

const showDropdown = () => {
  showOptions.value = true;
};
const hideDropdown = () => {
  setTimeout(() => {
    showOptions.value = false; 
  }, 100); 
};
const getProfileData = () => {
  return selectedField.value === "individual"
    ? invoice.userClientProfile.clientDataindividual
    : invoice.userClientProfile.clientDataOrganization;
};
const props = defineProps(["showHeader","width","createButton"]);
</script>



<template>
   <div >
    <div class="bg-white" :class="[!showHeader ? '' : 'hidden']">
      <Header
        :isLoader="isLoader"
        headerTitle="Client Profile"
        backButtonText="&nbsp &lt Back &nbsp  &nbsp "
        backRoute="Invoice"
        saveDraftButtonText=" &nbsp; &nbsp; Create &nbsp;&nbsp;"
        :saveDraftButtonColor="Colors.primary"
        :onSaveDraftButtonClick="handleSaveDraftButtonClick"
        :showDropdown="false"
        :showBackButton="false"
      />
    </div>
<div class="bg-gray-200 h-[max-content] p-4">
    <div :class="[width ? '!w-[100%]' : '']"
      class="modal-content max-h-full w-[100%] md:w-[90%] lg:w-[80%] xl:w-[70%] 2xl:w-[50%] justify-start"
    >
      <div class="flex">
        <div class="w-full p-8 bg-white">
          <div class="mb-4 flex ml-4">
            <label for="logoInput" class="">
              <div
                class="logo-placeholder border-none cursor-pointer hover:border-solid w-20 h-20 border-2 grid place-items-center text-slate-500 text-6xl font-bold"
              >
                <div v-if="isLoadingImg">
                  <a-space class="w-full">
                    <a-spin size="large" />
                  </a-space>
                </div>
                <div v-else>
                  <img
                    id="imagePreview"
                    :src="
                      selectedField === 'individual'
                        ? invoice.userClientProfile.clientDataindividual.url ||
                          'https://res.cloudinary.com/dfbsbullu/image/upload/v1709745593/iribv5nqn6iovph3buhe.png'
                        : invoice.userClientProfile.clientDataOrganization.url ||
                          'https://res.cloudinary.com/dfbsbullu/image/upload/v1709745593/iribv5nqn6iovph3buhe.png'
                    "
                    alt="Logo"
                    ref="logoPreview"
                    class="w-20 mb-4 h-20 p-2 cursor-pointer"
                  />
                  
                </div>
              </div>
              <input
                id="logoInput"
                type="file"
                accept="image/*"
                class=""
                style="display: none"
                @change="handleFileInputChange"
                ref="logoInputRef"
              />
            </label>

            <div class="flex-right w-48 ml-6">
              <table class="border border-black">
                <tr class="border border-black bg-gray-100">
                  <td>
                    <div
                      class="flex pl-4 cursor-pointer"
                      @click="selectField('individual')"
                    >
                      <p class="mb-1 mt-4 font-medium text-[14px]">
                        Individual
                      </p>
                    </div>
                  </td>
                  <td>
                    <span
                      v-if="selectedField === 'individual'"
                      class="text-orange-600 fa-solid fas fa-check mr-2"
                    ></span>
                  </td>
                </tr>
                <tr class="bg-gray-100">
                  <td>
                    <div
                      class="flex pl-4 cursor-pointer"
                      @click="selectField('organization')"
                    >
                      <p class="mb-1 mt-3 mr-12 font-medium text-[14px]">
                        Organization
                      </p>
                    </div>
                  </td>
                  <td>
                    <span
                      v-if="selectedField === 'organization'"
                      class="text-orange-600 fa-solid fas fa-check mr-2"
                    ></span>
                  </td>
                </tr>
              </table>
            </div>
          </div>
          <br />
          <div >
            <div class="mb-4">
              <hr class="mb-4" />
              <div v-if="selectedField === 'organization'" class="mb-2">
              <p class="justify-start flex font-medium text-[14px] ">
                <span class="text-[#ff0000]">*</span>Organization Name
                </p>
                <a-input
                  v-model:value="getProfileData().organizationName
                  "
                  type="text"
                  placeholder="First Name"
                  class="border p-2"
                />
              </div>
              <div class="grid grid-cols-2 gap-4">
                <div>
                  <p class="justify-start flex font-medium text-[14px]">
                    <span class="text-[#ff0000]">*</span>First Name
                  </p>
                  <a-input
                    v-model:value="
                      getProfileData().firstName
                    "
                    type="text"
                    placeholder="First Name"
                    class="w-full border p-2"
                  />
                </div>
                <div>
                  <p class="justify-start flex font-medium text-[14px]">
                    <span class="text-[#ff0000]">*</span>Last Name
                  </p>
                  <a-input
                    v-model:value="
                      getProfileData().lastName
                    "
                    type="text"
                    placeholder="Last Name"
                    class="w-full border p-2"
                  />
                </div>
              </div>
              <hr class="my-4" />
            </div>
            <div class="flex">
              <div class="w-1/2 pr-2">
                <p class="text-left ml-2 font-medium text-[14px]">Currency</p>
                <a-select
                  v-model:value="
                    getProfileData().currency
                  "
                  size="large"
                  class="w-full text-left"
                >
                  <a-select-option
                    v-for="currency in invoice.currencyOptions"
                    :key="currency.value"
                    :value="currency.value"
                  >
                    {{ currency.label }}
                  </a-select-option>
                </a-select>
              </div>

              <div class="w-1/2 pl-2">
                <p class="text-left ml-2 font-medium text-[14px]">Language</p>
                <a-select
                  v-model:value="
                    getProfileData().language
                  "
                  size="large"
                  class="w-full text-left"
                >
                  <a-select-option
                    v-for="language in invoice.languageOptions"
                    :key="language.value"
                    :value="language.value"
                  >
                    {{ language.label }}
                  </a-select-option>
                </a-select>
              </div>
            </div>
            <hr class="my-4" />
            <div>
              <p class="justify-start flex font-medium text-[14px]">
                <span class="text-[#ff0000]">*</span>Email Address
              </p>
              <a-input
                v-model:value="
                  getProfileData().email
                "
                type="text"
                placeholder="Email"
                class="w-full border p-2"
              />
            </div>
            <hr class="my-4" />
            <div>
              <p class="justify-start flex font-medium text-[14px]">
                <span class="text-[#ff0000]">*</span>Phone Number
              </p>
              <a-input
                v-model:value="
                  getProfileData().phone
                "
                type="text"
                placeholder="Phone Number"
                class="w-full border p-2"
              />
            </div>
            <hr class="my-4" />
            <div>
              <p class="justify-start flex font-medium text-[14px]">
                <span class="text-[#ff0000]">*</span>Address(Line 1)
              </p>
              <a-input
                v-model:value="
                  getProfileData().address1
                "
                type="text"
                placeholder="Streeet Line 1"
                class="w-full border p-2"
              />

              <a-input
                v-model:value="
                  getProfileData().address2
                "
                type="text"
                placeholder="Street Line 2"
                class="w-full mt-2 border p-2"
              />
            </div>
            <div class=" justify-between items-center grid grid-cols-2 mt-1 gap-2">
              <div class="">
                <p class="text-left font-medium text-[14px]">
                  <span class="text-[#ff0000]">*</span>Country
                </p>

                <div class="relative">
                  <a-input
                    v-model:value="
                      getProfileData().country
                    "
                    @focus="showDropdown"
                    @blur="hideDropdown"
                    @input="filterOptions"
                    placeholder="Select Country"
                    class="w-full"
                  />
                  <ul
                    v-show="showOptions"
                    class="dropdown absolute border rounded mt-2 overflow-y-auto flex gap-2 flex-col w-full bg-white text-left"
                  >
                    <li
                      v-for="option in filteredOptions"
                      :key="option.value"
                      @click="selectOption(option, $event)"
                      class="ml-2"
                    >
                      {{ option.label }}
                    </li>
                  </ul>
                </div>
              </div>
              <div class="">
                <p class="text-left ml-2 font-medium text-[14px]">
                  Postal Code
                </p>
                <a-input
                  v-model:value="
                    getProfileData().postalCode
                  "
                  type="number"
                  class=""
                  placeholder="Postal Code"
                />
              </div>
              <div class="">
                <p class="text-left font-medium text-[14px]"> <span class="text-[#ff0000]">*</span>State</p>
                <a-input
                  v-model:value="
                    getProfileData().state
                  "
                  type="text"
                  class=""
                  placeholder="State"
                />
              </div>
              <div class="">
                <p class="text-left ml-2 font-medium text-[14px]">City</p>
                <a-input
                  v-model:value="
                    getProfileData().city
                  "
                  placeholder="City"
                  type="text"
                  class=""
                />
              </div>
            </div>

            <hr class="my-4" />

            <div>
              <div  v-if="selectedField === 'organization'">
                <p class="justify-start flex font-medium text-[14px]">
                  Tax Identification Number
                </p>
                <a-input
                  v-model:value="
                    getProfileData().taxId
                  "
                  type="number"
                  class="w-full border p-2"
                />
              </div>
              <div>
                <p class="justify-start flex font-medium text-[14px] my-2">
                  Fax Number
                </p>
                <a-input
                  v-model:value="
                    getProfileData().faxNumber
                  "
                  type="number"
                  class="w-full border p-2"
                />
              </div>
              <hr class="my-4" />
              <div>
                <p class="justify-start flex font-medium text-[14px]">
                  Website URL
                </p>
                <a-input
                  v-model:value="
                    getProfileData().websiteURL
                  "
                  type="text"
                  class="w-full border p-2 mb-4"
                />
              </div>
              <div  v-if="selectedField === 'organization'">
                <p class="justify-start flex font-medium text-[14px]">Notes</p>
                <a-textarea
                  v-model:value="
                    getProfileData().notes
                  "
                  type="text"
                  class="w-full border p-2"
                />
              </div>
            </div>
          </div>

          <!-- <div v-else-if="selectedField === 'organization'" :key="2">
            <div class="mb-4">
              <hr class="mb-4" />
              <div class="flex flex-col">
                <p class="justify-start flex font-medium text-[14px]">
                  *Organization Name
                </p>
                <a-input
                  v-model:value="
                    invoice.userClientProfile.clientDataOrganization
                      .organizationName
                  "
                  type="text"
                  placeholder="First Name"
                  class="border p-2"
                />

                <div class="grid grid-cols-2 gap-4 mt-2">
                  <div>
                    <p class="justify-start flex font-medium text-[14px]">
                      <span class="text-[#ff0000]">*</span>First Name
                    </p>
                    <a-input
                      v-model:value="
                        invoice.userClientProfile.clientDataOrganization
                          .firstName
                      "
                      type="text"
                      placeholder="First Name"
                      class="w-full border p-2"
                    />
                  </div>
                  <div>
                    <p class="justify-start flex font-medium text-[14px]">
                      <span class="text-[#ff0000]">*</span>Last Name
                    </p>
                    <a-input
                      v-model:value="
                        invoice.userClientProfile.clientDataOrganization
                          .lastName
                      "
                      type="text"
                      placeholder="Last Name"
                      class="w-full border p-2"
                    />
                  </div>
                </div>
              </div>
            </div>
            <hr class="mb-4" />
            <div>
              <div class="flex">
                <div class="w-1/2 pr-2">
                  <p class="text-left ml-2 font-medium text-[14px]">Currency</p>
                  <a-select
                    v-model:value="
                      invoice.userClientProfile.clientDataOrganization.currency
                    "
                    size="large"
                    class="w-full text-left"
                  >
                    <a-select-option
                      v-for="currency in invoice.currencyOptions"
                      :key="currency.value"
                      :value="currency.value"
                    >
                      {{ currency.label }}
                    </a-select-option>
                  </a-select>
                </div>

                <div class="w-1/2 pl-2">
                  <p class="text-left ml-2 font-medium text-[14px]">Language</p>
                  <a-select
                    v-model:value="
                      invoice.userClientProfile.clientDataOrganization.language
                    "
                    size="large"
                    class="w-full text-left"
                  >
                    <a-select-option
                      v-for="language in invoice.languageOptions"
                      :key="language.value"
                      :value="language.label"
                    >
                      {{ language.label }}
                    </a-select-option>
                  </a-select>
                </div>
              </div>
              <hr class="my-4" />
              <p class="justify-start flex font-medium text-[14px]">
                <span class="text-[#ff0000]">*</span> Email Address
              </p>
              <a-input
                v-model:value="
                  invoice.userClientProfile.clientDataOrganization.email
                "
                type="text"
                placeholder="Email"
                class="w-full border p-2"
              />
            </div>
            <hr class="my-4" />
            <div>
              <p class="justify-start flex font-medium text-[14px]">
                <span class="text-[#ff0000]">*</span>Phone Number
              </p>
              <a-input
                v-model:value="
                  invoice.userClientProfile.clientDataOrganization.phone
                "
                type="text"
                placeholder="Phone Number"
                class="w-full border p-2"
              />
            </div>
            <hr class="my-4" />
            <div>
              <p class="justify-start flex font-medium text-[14px]">
                <span class="text-[#ff0000]">*</span>Address(Line 1)
              </p>
              <a-input
                v-model:value="
                  invoice.userClientProfile.clientDataOrganization.address1
                "
                type="text"
                placeholder="Streeet Line 1"
                class="w-full border p-2"
              />

              <a-input
                v-model:value="
                  invoice.userClientProfile.clientDataOrganization.address2
                "
                type="text"
                placeholder="Street Line 2"
                class="w-full mt-2 border p-2"
              />
            </div>
            <div class=" justify-between items-center grid grid-cols-2 mt-1 gap-2">
              <div class="">
                <p class="text-left font-medium text-[14px]">
                  <span class="text-[#ff0000]">*</span>Country
                </p>
                <div class="relative">
                  <a-input
                    v-model:value="
                      invoice.userClientProfile.clientDataOrganization.country
                    "
                    @focus="showDropdown"
                    @blur="hideDropdown"
                    @input="filterOptions"
                    placeholder="Select Country"
                    class="w-full"
                  />

                  <ul
                    v-show="showOptions"
                    class="dropdown absolute border rounded mt-2 overflow-y-auto w-full max-h-48 flex gap-2 flex-col bg-white text-left"
                  >
                    <li
                      v-for="option in filteredOptions"
                      :key="option.value"
                      @click="selectOption(option, $event)"
                      class="ml-2"
                    >
                      {{ option.label }}
                    </li>
                  </ul>
                </div>
              </div>
              <div class="">
                <p class="text-left ml-2 font-medium text-[14px]">
                  Postal Code
                </p>
                <a-input
                  v-model:value="
                    invoice.userClientProfile.clientDataOrganization.postalCode
                  "
                  type="number"
                  class=""
                  placeholder="Postal Code"
                />
              </div>
              <div class="">
                <p class="text-left font-medium text-[14px]"> <span class="text-[#ff0000]">*</span>State</p>
                <a-input
                  v-model:value="
                    invoice.userClientProfile.clientDataOrganization.state
                  "
                  type="text"
                  class=""
                  placeholder="State"
                />
              </div>
              <div class="">
                <p class="text-left ml-2 font-medium text-[14px]">City</p>
                <a-input
                  v-model:value="
                    invoice.userClientProfile.clientDataOrganization.city
                  "
                  placeholder="City"
                  type="text"
                  class=""
                />
              </div>
            </div>

            <hr class="my-4" />

            <div>
              <div>
                <p class="justify-start flex font-medium text-[14px]">
                  Tax Identification Number
                </p>
                <a-input
                  v-model:value="
                    invoice.userClientProfile.clientDataOrganization.taxId
                  "
                  type="number"
                  class="w-full border p-2"
                />
              </div>
              <hr class="my-4" />
              <div>
                <p class="justify-start flex font-medium text-[14px]">
                  Fax Number
                </p>
                <a-input
                  v-model:value="
                    invoice.userClientProfile.clientDataOrganization.faxNumber
                  "
                  type="number"
                  class="w-full border p-2"
                />
              </div>
              <hr class="my-4" />
              <div>
                <p class="justify-start flex font-medium text-[14px]">
                  Website URL
                </p>
                <a-input
                  v-model:value="
                    invoice.userClientProfile.clientDataOrganization.websiteURL
                  "
                  type="text"
                  class="w-full border p-2"
                />
              </div>
              <hr class="my-4" />
              <div>
                <p class="justify-start flex font-medium text-[14px]">Notes</p>
                <a-textarea
                  v-model:value="
                    invoice.userClientProfile.clientDataOrganization.notes
                  "
                  type="text"
                  class="w-full border p-2"
                />
              </div>
            </div>
          </div> -->
            <div class="flex justify-center  mt-4 "  :class="[createButton ? '' : 'hidden']">
            <div v-if="isLoader"><a-spin size="large"></a-spin></div>
            <a-button class="bg-blue-600 text-white w-64 h-12"
              v-else
              @click="handleSaveDraftButton()"
            >Create </a-button>
          </div>
          
        </div>
      </div>
    </div>
  </div>
  </div>
</template>

<style>
.dropdown {
  list-style-type: none;
  padding: 0;
  position: absolute;
  left: 0;
  z-index: 999;
}

.relative {
  position: relative;
}

.dropdown li {
  cursor: pointer;
}
</style>
