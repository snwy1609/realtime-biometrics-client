<template>
  <DataTable
    :rows="data['employees']??[]"
    :columns="columns"
    :cells="[
      'employee_id',
      'fullname',
      'contacts',
      'actions',
      'image',
      'position',
      'department',
    ]"

    :filter="search"
   
    
    v-if="data['employees']"
  >
    <!-- <template v-slot:image="{ props }">
      <q-td :props="props">
       
      </q-td>
    </template> -->

    <template v-slot:employee_id="{ props }">
      <q-td :props="props">
        <div class="column">
          <span>
            <span class="font-bold mr-1"> ID:</span>
            <span>{{ props.row.employee_id }}</span>
          </span>
          <span>
            <span class="font-bold mr-1">BID:</span>
            <span class="font-secondary">{{ props.row.biometrics_id }}</span>
          </span>
        </div>
      </q-td>
    </template>
    <template v-slot:fullname="{ props }">
      <q-td :props="props">
        <div class="row items-center w-fit">
          <q-avatar>
            <img :src="props.row.image" />
          </q-avatar>

          <div class="text-md px-2">{{ props.row.full_name }}</div>
        </div>
      </q-td>
    </template>
    <template v-slot:contacts="{ props }">
      <q-td :props="props">
        <div class="row bg-transparent justify-center">
          <q-icon
            name="mail"
            color="secondary"
            class="mx-1 cursor-pointer"
            size="1.5rem"
          >
            <q-tooltip anchor="top middle" self="top middle">
              {{ props.row.email }}
            </q-tooltip>
          </q-icon>
          <q-icon
            name="call"
            color="secondary"
            class="mx-1 cursor-pointer"
            size="1.5rem"
          >
            <q-tooltip anchor="top middle" self="top middle">
              {{ props.row.contact_number }}
            </q-tooltip>
          </q-icon>
        </div>
      </q-td>
    </template>

    <template v-slot:position="{ props }">
      <q-td :props="props">
        <q-chip
          size="md"
          v-for="position in props.row.positions"
          :key="position.id"
        >
          {{ position.name }}
        </q-chip>
      </q-td>
    </template>

    <template v-slot:department="{ props }">
      <q-td :props="props">
        <div>
          <q-chip
            size="md"
            v-for="department in props.row.departments"
            :key="department.id"
          >
            {{ department.name }}
          </q-chip>
        </div>
      </q-td>
    </template>

    <template v-slot:actions="{ props }">
      <q-td :props="props">
        <q-btn
          dense
          :to="{ name: 'employeeDetails', params: { id: props.row.id } }"
          class="w-28 text-xs"
          color="secondary"
          glossy
          @click="store.selectEmployee(props.row)"
          text-color="white"
          push
          label="View Details"
          icon="preview"
        />
      </q-td>
    </template>

    <template v-slot:top>
      <div class="row justify-between w-full items-center">
        <div class="row w-[600px] items-center">
          <div class="q-table__title">Employees</div>
        </div>

        <div class="row items-center">
          <!-- <SearchBar class="px-2" @search="search" /> -->
          <CreateEditEmployeeModal :title="'Add New Employees'">
          </CreateEditEmployeeModal>


          <ArchiveModal />

          <q-btn @click="attendanceStore.generateAllPDF()" label="Print Attendance" />
        </div>
      </div>

      <div class="row items-center  w-full justify-between">
        <div class="row  items-center">
          <span class="pr-3">Filter :</span>
        <SelectView
          class="m-2 ml-0"
          :data="departments"
          :label="'Department'"
          @onChange="filter($event, 'departments')"
        />

        <SelectView
          class="m-2 ml-0"
          :data="positions"
          :label="'Position'"
          @onChange="filter($event, 'positions')"
        />
        </div>

        <div>
          <q-input outlined label="Search" dense v-model="search" >
        <template v-slot:append>
          <q-icon name="search" />
        </template>
      </q-input>
         
        </div>
        
      </div>
    </template>
  </DataTable>
</template>
  
  <script>
import DataTable from "@/components/DataTable.vue";
import CreateEditEmployeeModal from "../modals/CreateEditEmployeeModal.vue";
import ArchiveModal from "../modals/ArchiveModal.vue";
import { useEmployeeStore } from "@/store/employee";
// import SearchBar from "@/components/SearchBar.vue";
import { onMounted, ref } from "vue";
import { storeToRefs } from "pinia";
import SelectView from "@/components/SelectView.vue";
import { useDepartmentStore } from "@/store/department";
import { usePositionStore } from "@/store/position";
import { useAttendanceStore } from '@/store/attendance';
const columns = [
  {
    name: "employee_id",
    required: true,
    label: " ID",
    align: "left",
    field: (row) => row.employee_id,
    // format: (val) => `${val}`,
    //  sortable: true,
  },
  {
    name: "fullname",
    required: true,
    label: "Employee Name",
    align: "left",
    field: (row) => row.full_name,
    // format: (row) =>
    //   `${row.lastname}, ${row.firstname} ${
    //     row.middlename ? row.middlename[0] : ""
    //   }.`,
    sortable: true,
  },
  {
    name: "contacts",
    required: true,
    label: "Contacts",
    align: "center",
  },
  {
    name: "position",
    required: true,
    label: "Position",
    align: "left",
  },
  {
    name: "department",
    required: true,
    label: "Department",
    align: "center",
  },
  {
    name: "address",
    required: true,
    label: "Address",
    align: "left",
    field: (row) => row.address,
    format: (val) => `${val}`,
  },
  {
    name: "actions",
    required: true,
    label: "Actions",
    align: "center",
  },
];

export default {
  components: {
    DataTable,
   
    CreateEditEmployeeModal,
    ArchiveModal,
    SelectView,
  },
  setup() {
    const store = useEmployeeStore();
    const attendanceStore = useAttendanceStore()

    const departmentStore = useDepartmentStore();
    const positionStore = usePositionStore();
    const { departments } = storeToRefs(departmentStore);
    const { positions } = storeToRefs(positionStore);
    const { data } = storeToRefs(store);

    const loading = ref([]);

    onMounted(() => {
      store.getEmployees();
    });

    return {
      columns,
      departments,
      positions,
      data,
      store,
      loading,
      onDelete: async (employee_id) => {
        loading.value[employee_id] = true;
        await new Promise((resolve) => setTimeout(resolve, 1000));
        await store.destroy(employee_id);
        loading.value[employee_id] = false;
      },

      onChangePage: (page) => {
        console.log(data.value["employees"].links[page].url);
        console.log(page);

        let name = "employees";

        store.paginate(name, data.value[name].links[page].url);
      },
      filter: async (val, attribute) => {
        if (val) {
          store.filter(attribute, val.id);
        }
      },
      // search: (val) => {
      //   store.search(val);
      // },
      paginate:(val)=>{
        store.paginate('employees',val)

      },
      search:ref('') ,
      attendanceStore
    };
  },
};
</script>
  
  <style>
</style>