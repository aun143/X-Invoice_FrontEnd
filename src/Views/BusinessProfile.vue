<script setup>
import { ref, onMounted } from "vue";
import Header from "../components/Header.vue";
import { Colors } from "../utils/color";
import { useRouter, useRoute } from "vue-router";
import { useInvoiceStore } from "../stores/index";
import {
  PatchBusinessProfilerIndiviualApi,
  PatchBusinessProfilerOrganizationApi,
} from "../service/BusinessProfileService";
import { getUserDetailsApi } from "../service/LoginService";
import { uploadImage } from "../service/UploadImage";
import Button from "../components/Button.vue";
import { notification } from "ant-design-vue";
const isLoading = ref(false);
const isLoader = ref(false);
const isLoadingImg = ref(false);
const route = useRoute();
const router = useRouter();
const invoice = useInvoiceStore();
const firstNameError = ref("");
const lastNameError = ref("");
const emailError = ref("");
const address1Error = ref("");
const address2Error = ref("");
const cityError = ref("");

const validateForm = (profileType) => {
  // Reset errors
  firstNameError.value = "";
  lastNameError.value = "";
  emailError.value = "";
  address1Error.value = "";
  address2Error.value = "";
  cityError.value = "";

  const profileData =
    profileType === "individual"
      ? invoice.userProfileData.individualProfile
      : invoice.userProfileData.organizationProfile;

  // Validate First Name
  if (!/^[a-z A-Z]+$/.test(profileData.firstName)) {
    firstNameError.value =
      "First name must contain only letters from A-Z and a-z";
  }
  if (!/^[a-z A-Z]+$/.test(profileData.lastName)) {
    lastNameError.value =
      "Last name must contain only letters from A-Z and a-z";
  }
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(profileData.email)) {
    emailError.value = "Email must be Valid and contain '@' ";
  }
  if (!/^[a-z A-Z ]+$/.test(profileData.city)) {
    cityError.value = "City must contain only Alphabetic A-Z characters";
  }

  // Check if any errors
  if (
    firstNameError.value ||
    lastNameError.value ||
    emailError.value ||
    address1Error.value ||
    address2Error.value ||
    cityError.value
  ) {
    return false;
  }

  return true;
};
const openNotificationWithIcon = (type, message) => {
  notification[type]({
    message: type === "success" ? "Success" : "Error",
    description: message,
    duration: 3,
  });
};
const submitbusinessProfileDataOrganization = async (id) => {
  if (isLoadingImg.value) {
    openNotificationWithIcon("error", "Please Wait To upload Image First");
    return;
  }
  try {
    isLoader.value = true;
    const { success, data, error } = await PatchBusinessProfilerOrganizationApi(
      invoice.userProfileData.organizationProfile._id,
      invoice.userProfileData.organizationProfile
    );

    if (success) {
      invoice.updateUserProfileAndBusinessProfile(data);
      router.push("/");
      openNotificationWithIcon(
        "success",
        data || "Profile has been Updated successfully."
      );
    } else {
      console.error("Error During Profile Updation:", error);
      openNotificationWithIcon(
        "error",
        error || "An error occurred while Updating the Profile."
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
    console.error("Error During Profile Updation:", error);
    openNotificationWithIcon(
      "error",
      "An error occurred while Updating the Profile."
    );
  } finally {
    isLoader.value = false;
  }
};

const submitbusinessProfileDataindividual = async (Id) => {
  if (isLoadingImg.value) {
    openNotificationWithIcon("error", "Please Wait To upload Image First");
    return;
  }
  try {
    isLoader.value = true;
    const { success, data, error } = await PatchBusinessProfilerIndiviualApi(
      invoice.userProfileData.individualProfile._id,
      invoice.userProfileData.individualProfile
    );

    if (success) {
      router.push("/");
      invoice.updateUserProfileAndBusinessProfile(data);
      openNotificationWithIcon(
        "success",
        data || "Profile has been Updated successfully."
      );
    } else {
      console.error("Error During Profile individual:", error);
      openNotificationWithIcon(
        "error",
        error || "An error occurred while Updating the Profile."
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
    console.error("Error During Profile individual:", error);
    openNotificationWithIcon(
      "error",
      "An error occurred while Updating the Profile."
    );
  } finally {
    isLoader.value = false;
  }
};

onMounted(async () => {
  try {
    isLoading.value = true;

    const UserId = localStorage.getItem("UserId");

    if (UserId) {
      const UserResponse = await getUserDetailsApi(UserId);
      invoice.userProfileData = UserResponse;
      invoice.updateUser(invoice.userProfileData);

      const { userProfileData } = invoice;

      if (userProfileData.selectedProfileType === "individual") {
        invoice.updateUserProfileIndividual(userProfileData.individualProfile);

        // Display the uploaded image URL if available
        if (invoice.userProfileData.individualProfile.url) {
          displayImage(
            logoInputRef.value,
            invoice.userProfileData.individualProfile.url
          );
        }
      } else if (userProfileData.selectedProfileType === "organization") {
        invoice.updateUserProfileOrganization(
          userProfileData.organizationProfile
        );

        // Display the uploaded image URL if available
        if (invoice.userProfileData.organizationProfile.url) {
          displayImage(
            logoInputRef.value,
            invoice.userProfileData.organizationProfile.url
          );
        }
      }
    } else {
      router.push("/login");
    }
  } catch (error) {
    openNotificationWithIcon(
      "error",
      error || "An error occurred while getting the Profiles."
    );

    console.error("Error During Account Update:", error);
  } finally {
    isLoading.value = false;
  }
});

const profileType = ref(invoice.selectedProfileType);
const switchProfileType = (type) => {
  invoice.selectProfileType(type);
  profileType.value = type;
  //console.log(`Selected profile type: ${type}`);
};

const handleSaveDraftButtonClick = () => {
  if (profileType.value === "individual") {
    if (validateForm("individual")) {
      submitbusinessProfileDataindividual();
    }
  } else if (profileType.value === "organization") {
    if (validateForm("organization")) {
      submitbusinessProfileDataOrganization();
    }
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
        if (profileType.value === "individual") {
          invoice.userProfileData.individualProfile.url = imageUrl;
        } else if (profileType.value === "organization") {
          invoice.userProfileData.organizationProfile.url = imageUrl;
        }
        displayImage(logoInputRef.value, imageUrl); // Update the image preview
      } else {
        console.error("Failed To Upload File");
      }
    } catch (error) {
      console.error("Error uploading image:", error);
      openNotificationWithIcon(
        "error",
        error || "Error uploading image. Please try again."
      );
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
      if (profileType.value === "individual") {
        invoice.userProfileData.individualProfile.url = imageUrl;
        console.log("invoice.ind.url", imageUrl);
      } else if (profileType.value === "organization") {
        invoice.userProfileData.organizationProfile.url = imageUrl;
        console.log("invoice.org.url", imageUrl);
      } // Update the URL in the formData
      const imagePreview = document.getElementById("imagePreview");
      imagePreview.src = imageUrl;
    };

    reader.readAsDataURL(file);
    if (profileType.value === "individual") {
      invoice.userProfileData.individualProfile.url = file;
      console.log("invoice.ind.url", file);
    } else if (profileType.value === "organization") {
      invoice.userProfileData.organizationProfile.url = file;
      console.log("invoice.org.url", file);
    }
  }
};
const getProfileData = () => {
  return profileType.value === "individual"
    ? invoice.userProfileData.individualProfile
    : invoice.userProfileData.organizationProfile;
};
const addingCustomField = ref(false);
const toggleAddingCustomField = () => {
  if (!addingCustomField.value) {
    getProfileData().customFields.push({
      customFieldName: "",
      customFieldValue: ""
    });
  } else {
    getProfileData().customFields.pop();
  }
  addingCustomField.value = !addingCustomField.value;
};
  const toggleCustomField = (index) => {
  if (index === 0) {
    getProfileData().customFields.push({
      customFieldName: "",
      customFieldValue: ""
    });
  } else {
    getProfileData().customFields.splice(index, 1);
  }
};
</script>

<template>
  <div v-if="isLoading" class="flex justify-center items-center">
    <a-space class="w-full flex h-96 justify-center items-center">
      <a-spin size="large" />
    </a-space>
  </div>

  <div v-else>
    <div class="bg-white">
      <Header
        :isLoader="isLoader"
        headerTitle="Business Profile"
        backButtonText="&nbsp &lt Back &nbsp  &nbsp"
        backRoute="Invoice"
        saveDraftButtonText=" Save Changes"
        :saveDraftButtonColor="Colors.primary"
        :onSaveDraftButtonClick="handleSaveDraftButtonClick"
        :showDropdown="false"
        :showDraftButton="false"
        :showBackButton="false"
      />
    </div>
    <div class="bg-gray-200 h-[max-content]">
      <div
        class="flex container pt-4 p-4 w-[100%] md:w-[90%] lg:w-[80%] xl:w-[70%] 2xl:w-[50%] justify-start"
      >
        <div class="w-full p-8 bg-white">
          <div class="flex ml-4">
            <label for="logoInput" class="">
              <div
                class="logo-placeholder hover:border-dashed border-none cursor-pointer w-20 h-20 border-2 grid place-items-center text-slate-500 text-6xl font-bold"
              >
                <div v-if="isLoadingImg">
                  <a-space class="w-full">
                    <a-spin size="large" />
                  </a-space>
                </div>
                <div v-else>
                  <img
                    v-if="!isLoadingImg"
                    id="imagePreview"
                    :src="
                      profileType === 'individual'
                        ? invoice.userProfileData.individualProfile.url ||
                          'https://res.cloudinary.com/dfbsbullu/image/upload/v1709745593/iribv5nqn6iovph3buhe.png'
                        : invoice.userProfileData.organizationProfile.url ||
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
                <tr class="border-black border">
                  <td>
                    <div
                      class="flex pl-4 cursor-pointer bg-gray-100"
                      @click="switchProfileType('individual')"
                    >
                      <p class="mb-1 mt-4 mr-12 font-medium text-[14px]">
                        Individual
                      </p>
                    </div>
                  </td>
                  <td class="bg-gray-100">
                    <span
                      v-if="profileType === 'individual'"
                      class="text-orange-600 fa-solid fas fa-check mr-2"
                    ></span>
                  </td>
                </tr>
                <tr>
                  <td>
                    <div
                      class="flex pl-4 cursor-pointer bg-gray-100"
                      @click="switchProfileType('organization')"
                    >
                      <p class="mb-1 mt-3 mr-12 font-medium text-[14px]">
                        Organization
                      </p>
                      <!-- <span v-if="selectedField === 'organization'" class="text-orange-500">&#10003;</span> -->
                    </div>
                  </td>
                  <td class="bg-gray-100">
                    <span
                      v-if="profileType === 'organization'"
                      class="text-orange-600 fa-solid fas fa-check mr-2"
                    ></span>
                  </td>
                </tr>
              </table>
            </div>
          </div>
          <!-- <div class="flex">
          Looking to Change your account Logo and Branding?
          <a @click="logo" class="cursor-pointer"> Account Customization</a>
        </div> -->
          <hr class="my-4" />
          <br />
          <div>
            <div class="mb-4">
              <div class="bg-[lightgray] py-2">
                <label class="flex font-bold mb-2 ml-4 mt-2"
                  >Personal Information
                </label>
              </div>
              <div class="mt-4" v-if="profileType === 'organization'">
                <p class="justify-start flex font-medium text-[14px]">
                  Organization Name
                </p>
                <a-input
                  v-model:value="getProfileData().organizationName"
                  type="text"
                  placeholder="Organization Name"
                  class="w-full p-2"
                />
              </div>
              <div class="grid grid-cols-2 mt-4 gap-4">
                <div>
                  <p class="justify-start flex font-medium text-[14px]">
                    <span class="text-[#ff0000]">*</span>First Name
                  </p>
                  <a-input
                    v-model:value="getProfileData().firstName"
                    type="text"
                    placeholder="First Name"
                    class="w-full p-2"
                  />
                  <p v-if="firstNameError" class="text-red-500">
                    {{ firstNameError }}
                  </p>
                </div>
                <div>
                  <p class="justify-start flex font-medium text-[14px]">
                    <span class="text-[#ff0000]">*</span>Last Name
                  </p>
                  <a-input
                    v-model:value="getProfileData().lastName"
                    type="text"
                    placeholder="Last Name"
                    class="w-full p-2"
                  />
                  <p v-if="lastNameError" class="text-red-500">
                    {{ lastNameError }}
                  </p>
                </div>
                <div>
                  <p class="justify-start flex font-medium text-[14px]">
                    <span class="text-[#ff0000]">*</span>Email Address
                  </p>
                  <a-input
                    v-model:value="getProfileData().email"
                    type="email"
                    placeholder="Email"
                    class="w-full p-2"
                  />
                  <p v-if="emailError" class="text-red-500">{{ emailError }}</p>
                </div>
                <div>
                  <p class="justify-start flex font-medium text-[14px]">
                    Website URL
                  </p>
                  <a-input
                    v-model:value="getProfileData().websiteURL"
                    type="text"
                    placeholder="Website URL"
                    class="w-full p-2"
                  />
                </div>
              </div>
            </div>

            <div class="">
              <div class="bg-[lightgray] py-2">
                <label class="flex font-bold mb-2 ml-4 mt-2 c">
                  Contact Details</label
                >
              </div>
              <div class="grid grid-cols-2 mt-4 gap-4">
                <div class="col-span-2">
                  <div class="flex">
                    <div class="w-1/2 pr-2">
                      <p class="justify-start flex font-medium text-[14px]">
                        <span class="text-[#ff0000]">*</span>Address(Line 1)
                      </p>
                      <a-input
                        v-model:value="getProfileData().address1"
                        type="text"
                        placeholder="Address"
                        class="w-full p-2"
                      />
                      <p v-if="address1Error" class="text-red-500">
                        {{ address1Error }}
                      </p>
                    </div>
                    <div class="w-1/2 pl-2">
                      <p class="justify-start flex font-medium text-[14px]">
                        Address (Line 2)
                      </p>
                      <a-input
                        v-model:value="getProfileData().address2"
                        type="text"
                        placeholder="Address2"
                        class="w-full p-2"
                      />
                      <p v-if="address2Error" class="text-red-500">
                        {{ address2Error }}
                      </p>
                    </div>
                  </div>

                  <div class="flex mt-4">
                    <div class="w-1/2 pr-2">
                      <p class="justify-start flex font-medium text-[14px]">
                        Postal Code
                      </p>
                      <a-input
                        v-model:value="getProfileData().postalCode"
                        type="number"
                        class="w-full border p-2"
                      />
                    </div>
                    <div class="w-1/2 pl-2">
                      <p class="justify-start flex font-medium text-[14px]">
                        State
                      </p>
                      <a-input
                        v-model:value="getProfileData().state"
                        type="text"
                        class="w-full border p-2"
                      />
                    </div>
                  </div>
                  <div class="flex mt-4">
                    <div class="w-1/2 pr-2">
                      <p class="justify-start flex font-medium text-[14px]">
                        <span class="text-[#ff0000]">*</span>City
                      </p>
                      <a-input
                        v-model:value="getProfileData().city"
                        type="text"
                        class="w-full border p-2"
                      />
                      <p v-if="cityError" class="text-red-500">
                        {{ cityError }}
                      </p>
                    </div>
                    <div class="w-1/2 pl-2">
                      <p class="justify-start flex font-medium text-[14px]">
                        <span class="text-[#ff0000]">*</span>Country
                      </p>
                      <a-select
                        v-model:value="getProfileData().country"
                        size="large"
                        class="w-full text-left"
                      >
                        <a-select-option
                          v-for="country in invoice.countryOptions"
                          :key="country.value"
                          :value="country.value"
                        >
                          {{ country.label }}
                        </a-select-option>
                      </a-select>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <div class="my-4 flex flex-col">
              <div class="bg-[lightgray] py-2">
                <label class="flex font-bold mb-2 ml-4 mt-2">
                  Additional Information
                </label>
              </div>
              <div class="grid grid-cols-2 mt-4 gap-4 mb-2">
                <div>
                  <p class="justify-start flex font-medium text-[14px]">
                    Phone Number
                  </p>

                  <a-input
                    v-model:value="getProfileData().phone"
                    type="text"
                    class="w-full border p-2"
                  />
                </div>
                <div>
                  <p class="justify-start flex font-medium text-[14px]">
                    Fax Number
                  </p>
                  <a-input
                    v-model:value="getProfileData().faxNumber"
                    type="number"
                    class="w-full border p-2"
                  />
                </div>
                <div>
                  <p class="justify-start flex font-medium text-[14px]">
                    Tax Identification Number
                  </p>
                  <a-input
                    v-model:value="getProfileData().taxId"
                    type="number"
                    class="w-full border p-2"
                  />
                </div>
                <div>
                  <p class="justify-start flex font-medium text-[14px]">
                    Notes
                  </p>
                  <a-textarea
                    v-model:value="getProfileData().notes"
                    type="text"
                    class="w-full p-2"
                  />
                </div>
              </div>

              <hr class="my-4" />
              <div
      class="grid grid-cols-2 gap-4"
      v-for="(customField, index) in getProfileData().customFields"
      :key="index"
    >
      <div>
        <p class="justify-start flex font-medium text-[14px]">
          Custom Field
        </p>
        <a-input
          v-model:value="customField.customFieldName"
          type="text"
          class="w-full p-2"
          placeholder="Custom Field Name"
        />
      </div>
      <div>
        <p
          @click="toggleCustomField(index)"
          class="justify-end text-blue-600 cursor-pointer flex text-[14px]"
        >
          {{ index === 0 ? 'Add Custom Field' : 'Remove Custom Field' }}
        </p>
        <a-input
          v-model:value="customField.customFieldValue"
          type="text"
          class="w-full p-2"
          placeholder="Custom Field Value"
        />
      </div>
    </div>
            </div>
          </div>
          <!-- <div v-else-if="profileType === 'organization'" :key="2">
          <div class="">
            <div class="bg-[lightgray] py-2">
              <label class="flex font-bold ml-4 my-2 c">
                Personal Information
              </label>
            </div>
            <div class=" mt-4">
              <p class="justify-start flex font-medium text-[14px]">Organization Name</p>
              <a-input
                v-model:value="
                  invoice.userProfileData.organizationProfile.organizationName
                "
                type="text"
                placeholder="Organization Name"
                class="w-full p-2 mb-4"
              />
            </div>
            <div class="grid grid-cols-2 gap-4">
              <div>
                <p class="justify-start flex font-medium text-[14px]">
                  <span class="text-[#ff0000]">*</span>First Name
                </p>
                <a-input
                  v-model:value="
                    invoice.userProfileData.organizationProfile.firstName
                  "
                  type="text"
                  placeholder="First Name"
                  class="w-full p-2"
                />
                <p v-if="firstNameError" class="text-red-500">
                  {{ firstNameError }}
                </p>
              </div>
              <div>
                <p class="justify-start flex font-medium text-[14px]">
                  <span class="text-[#ff0000]">*</span>Last Name
                </p>
                <a-input
                  v-model:value="
                    invoice.userProfileData.organizationProfile.lastName
                  "
                  type="text"
                  placeholder="Last Name"
                  class="w-full p-2"
                />
                <p v-if="lastNameError" class="text-red-500">
                  {{ lastNameError }}
                </p>
              </div>
              <div>
                <p class="justify-start flex font-medium text-[14px]">
                  <span class="text-[#ff0000]">*</span>Email Address
                </p>
                <a-input
                  v-model:value="
                    invoice.userProfileData.organizationProfile.email
                  "
                  type="text"
                  placeholder="Email"
                  class="w-full p-2"
                />
                <p v-if="emailError" class="text-red-500">{{ emailError }}</p>
              </div>
              <div>
                <p class="justify-start flex font-medium text-[14px]">Website URL</p>
                <a-input
                  v-model:value="
                    invoice.userProfileData.organizationProfile.websiteURL
                  "
                  type="text"
                  placeholder="Website URL"
                  class="w-full p-2"
                />
              </div>
            </div>
          </div>

          <div class="my-4 flex flex-col">
            <div class="bg-[lightgray] py-2">
              <label class="flex font-bold mb-2 ml-4 mt-2 c"> Contact Details</label>
            </div>
            <div class="grid grid-cols-2  mt-4 gap-4">
              <div class="col-span-2">
                <div class="flex">
                  <div class="w-1/2 pr-4">
                    <p class="justify-start flex font-medium text-[14px]">
                      <span class="text-[#ff0000]">*</span>Address(Line 1)
                    </p>
                    <a-input
                      v-model:value="
                        invoice.userProfileData.organizationProfile.address1
                      "
                      type="text"
                      placeholder="Address"
                      class="w-full p-2"
                    />
                    <p v-if="address1Error" class="text-red-500">
                      {{ address1Error }}
                    </p>
                  </div>
                  <div class="w-1/2">
                    <p class="justify-start flex font-medium text-[14px]">Address (Line 2)</p>
                    <a-input
                      v-model:value="
                        invoice.userProfileData.organizationProfile.address2
                      "
                      type="text"
                      placeholder="Address 2"
                      class="w-full p-2"
                    />
                    <p v-if="address2Error" class="text-red-500">
                      {{ address2Error }}
                    </p>
                  </div>
                </div>
                <div class="flex  my-4">
                  <div class="w-1/2 pr-4">
                    <p class="justify-start flex font-medium text-[14px]">Postal Code</p>
                    <a-input
                      v-model:value="
                        invoice.userProfileData.organizationProfile.postalCode
                      "
                      type="number"
                      class="w-full p-2"
                    />
                  </div>
                  <div class="w-1/2">
                    <p class="justify-start flex font-medium text-[14px]">State</p>
                    <a-input
                      v-model:value="
                        invoice.userProfileData.organizationProfile.state
                      "
                      type="text"
                      class="w-full p-2"
                    />
                  </div>
                </div>
                <div class="flex ">
                  <div class="w-1/2 pr-4">
                    <p class="justify-start flex font-medium text-[14px]">
                      <span class="text-[#ff0000]">*</span>City
                    </p>
                    <a-input
                      v-model:value="
                        invoice.userProfileData.organizationProfile.city
                      "
                      type="text"
                      class="w-full p-2"
                    />
                    <p v-if="cityError" class="text-red-500">{{ cityError }}</p>
                  </div>
                  <div class="w-1/2">
                    <p class="justify-start flex font-medium text-[14px]">
                      <span class="text-[#ff0000]">*</span>Country
                    </p>
                    <a-select
                      style="text-align: left"
                      v-model:value="
                        invoice.userProfileData.organizationProfile.country
                      "
                      size="large"
                      class="w-full"
                    >
                      <a-select-option
                        v-for="country in invoice.countryOptions"
                        :key="country.value"
                        :value="country.value"
                      >
                        {{ country.label }}
                      </a-select-option>
                    </a-select>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="mb-">
            <div class="bg-[lightgray] py-2">
              <label class="flex font-bold mb-2 ml-4 mt-2 c">
                Additional Information
              </label>
            </div>
            <div class="grid grid-cols-2  mt-4 gap-4 mb-2">
              <div>
                <p class="justify-start flex font-medium text-[14px]">Phone Number</p>
                <a-input
                  v-model:value="
                    invoice.userProfileData.organizationProfile.phone
                  "
                  type="text"
                  class="w-full p-2"
                />
              </div>
              <div>
                <p class="justify-start flex font-medium text-[14px]">Fax Number</p>
                <a-input
                  v-model:value="
                    invoice.userProfileData.organizationProfile.faxNumber
                  "
                  type="number"
                  class="w-full p-2"
                />
              </div>
              <div>
                <p class="justify-start flex font-medium text-[14px]">Tax Identification Number</p>
                <a-input
                  v-model:value="
                    invoice.userProfileData.organizationProfile.taxId
                  "
                  type="number"
                  class="w-full p-2"
                />
              </div>
              <div>
                <p class="justify-start flex font-medium text-[14px]">Notes</p>
                <a-textarea
                  v-model:value="
                    invoice.userProfileData.organizationProfile.notes
                  " 
                  type="text"
                  class="w-full p-2"
                />
              </div>
            </div>
            <hr class="my-4" />
            <div class="grid grid-cols-2 gap-4 ">
              <div>
                <p class="justify-start flex font-medium text-[14px]">Custom Field</p>
                <a-input
                  v-model:value="
                    invoice.userProfileData.organizationProfile.customFieldName
                  "
                  type="text"
                  class="w-full p-2"
                  placeholder="Custom Field Name"
                />
              </div>
              <div>
                <a-input
                  v-model:value="
                    invoice.userProfileData.organizationProfile.customFieldValue
                  "
                  type="text"
                  class="w-full p-2 mt-5"
                  placeholder="Custom Field Value"
                />
              </div>
            </div>
          </div>
        </div> -->
          
        <div class="flex justify-center mt-8">
            <div v-if="isLoader"><a-spin size="large"></a-spin></div>
            <a-button
              v-else
              buttonText="Save Changes"
              class="h-12 w-64 bg-blue-600 text-white"
              @click="handleSaveDraftButtonClick()"
            >Save Changes</a-button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.ant-select-selection {
  background-color: #15bb15;
}
</style>
