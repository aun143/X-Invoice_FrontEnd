
<script setup>
import Swal from "sweetalert2";
import VueSidebarMenuAkahon from "vue-sidebar-menu-akahon";
import { useRoute, useRouter } from "vue-router";
import { getUserDetailsApi } from "../src/service/LoginService";
import { computed, ref, onMounted, watch } from "vue";
import { useInvoiceStore } from "../src/stores/index";
const menuItems = [
  { link: "/dashBoard", name: "DashBoard", icon: "bxs-dashboard" },
  { link: "/Index", name: "Invoices", icon: "bxs-inbox" },
  { link: "/AllClients", name: "All Clients", icon: "bxs-user-detail" },
  { link: "/businessProfile", name: "Business", icon: "bxs-user-account" },
];

const router = useRouter();
const invoice = useInvoiceStore();

const handleExitButtonClick = async () => {
  const result = await Swal.fire({
    title: "Logout Confirmation",
    text: "Are you sure you want to log out?",
    icon: "warning",
    showCancelButton: true,
    confirmButtonColor: "#3085d6",
    cancelButtonColor: "#d33",
    confirmButtonText: "Yes, log me out!",
  });

  if (result.isConfirmed) {
    localStorage.removeItem("UserId");
    localStorage.removeItem("accessToken");
    localStorage.removeItem("userRole");
    invoice.userProfileData.userRole = "";
    router.push({ name: "Login" });
  }
};

const enableExitButton = true;
const isLoading = ref(false);
const disableSearch = false;
const componentsWithSidebar = [
  "Index",
  "Client",
  "AllClients",
  "Invoice",
  "BusinessProfile",
  "ViewClient",
  "GetInvoice",
  "EditInvoice",
  "EditClient",
  "SendInvoice",
  "Subscription",
  "DashBoard",
];
const UserRole=async()=>{
  try {
    isLoading.value = true;

    const UserId = localStorage.getItem("UserId");

    if (UserId) {
      const userProfileData = await getUserDetailsApi(UserId);
      invoice.userProfileData.userRole = userProfileData.userRole;
    } else {
      router.push("/login");
    }
    // showPopup();
  } catch (error) {
    Swal.fire({
      icon: "error",
      title: "Oops...",
      text: ("Error During Getting Account User Role:", error),
      footer: "Please try again ",
    });
    console.error("Error During getting Getting User Role:", error);
  } finally {
    isLoading.value = false;
  }
}
onMounted(async () => {
  UserRole();
});

const route = useRoute();
const enableSidebar = computed(() => {
  if (
    invoice.userProfileData.userRole === undefined ||
    invoice.userProfileData.userRole === ""
  ) {
    return false;
  }
  return componentsWithSidebar.includes(route.name);
});

const menuLogo =
  "https://res.cloudinary.com/dfbsbullu/image/upload/v1709745593/iribv5nqn6iovph3buhe.png";
const profileImg =
  "https://res.cloudinary.com/dfbsbullu/image/upload/v1709745593/iribv5nqn6iovph3buhe.png";

const upgradeAccount = () => {
  router.push("/subscription");
};

const online = ref(navigator.onLine);
watch(online, (value) => {
  if (!value) {
  //   Swal.fire({
  //     icon: "error",
  //     title: "No Internet Connection",
  //     text: "Please check your internet connection and try again.",
  //     showConfirmButton: false,
  //     allowOutsideClick: false, // Prevent closing when clicking outside
  //     didOpen: () => {
  //       const interval = setInterval(() => {
  //         if (navigator.onLine) {
  //           Swal.close();
  //           clearInterval(interval);
  //         }
  //       }, 1000); 
  //     },
  //   });
  }
});

window.addEventListener("online", () => {
  online.value = true;
  // window.location.reload();
});

window.addEventListener("offline", () => {
  online.value = false;
});
</script>
<template>
  <div class="overflow-hidden hidden md:block" :style="{ 'pointer-events': online ? 'auto' : 'none' }">
    <div class="">
      <VueSidebarMenuAkahon
        v-if="enableSidebar && online"
        menuTitle="X-Invoice"
        :menuLogo="menuLogo"
        :profileImg="profileImg"
        bgColor="#4AA7AD"
        profileName="Logout"
        :menuItems="menuItems"
        :isSearch="disableSearch"
        :isExitButton="enableExitButton"
        @button-exit-clicked="false"
        :isUsedVueRouter="true"
      >
      </VueSidebarMenuAkahon>
    </div>
    <RouterView
      v-if="enableSidebar"
      :style="{ paddingLeft: !online ? '0 !important' : '' }"
      class="max-w-full pb-6 pl-[250px] lg:pl-[250px] md:pl-[195px] "
    />
    <RouterView v-else />
    <div v-if="!online" class="offline-message">
      <p>No Internet Connection</p>
      
      <!-- You can style this message as needed -->
    </div>
    <div class="fixed bottom-0 w-full left-0  z-[999]" v-if="enableSidebar && online">
      <!-- <div v-if="invoice.userProfileData.userRole !== 'iSuperAdmin'"> -->
        <!-- v-if="invoice.userProfileData.userRole"-->
          <button
          @click="handleExitButtonClick" 
          type="primary"
  class="flex items-center text-red-600 py-8 justify-center h-[40px] rounded ] cursor-pointer trans  shadow-lg "
>
  <span class="mx-4 text-lg "><i class="fas fa-user mr-2"></i>Logout</span>
</button>
<!-- </div> -->
    </div>
  </div>
  <div class="block md:hidden overflow-auto  justify-center items-center bg-[#10C0CB] text-white  p-32">
  <h1>  For a better experience, we suggest using this website on a horizontal or larger screen. Thank you!

please replace the error message on mobile screens with this
Thanks</h1>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
.sidebar.open .profile #log_out{
  width:100%;
  background: #10C0CB;
}
@media (max-width: 1023px) {
  .sidebar {
    width: 195px !important;
  }
  .trans {
  box-shadow: 0px 2px 6px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s ease;
  width:195px !important;  
}
}
@media (max-width: 1023px) {
  .sidebar div.profile {
    width: 195px !important;
  }
  .trans {
  box-shadow: 0px 2px 6px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s ease;
  width:195px !important;  

}
}
.sidebar div.profile  {
background-color: transparent !important;
  }

  .sidebar div.profile {
    display: none !important;
  }

button:focus {
  outline: none;
}
.sidebar {
  transition: none !important;
}
.trans {
  box-shadow: 0px 2px 6px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s ease;
  width:250px;  

}
..trans {
  box-shadow: 0px 2px 6px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s ease;
}
#btn {
  display: none;
}
.float-button:hover {
  background-color: #388d94;
}
.offline-message {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  background-color: rgba(255, 0, 0, 0.8);
  color: white;
  text-align: center;
  padding: 10px;
  z-index: 9999; 
}
</style>
