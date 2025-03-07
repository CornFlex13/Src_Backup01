<script setup>
import { ref } from 'vue';
import { collection, getDocs, addDoc, deleteDoc, doc } from 'firebase/firestore';
import { db } from '../firebase'; // แก้ตาม path ที่คุณระบุ

const classPeriods = ref([]);
const dialog = ref(false);
const deleteDialog = ref(false);
const periodName = ref('');
const date = ref('');
const time = ref('');
const currentDeleteId = ref('');

// ฟังก์ชันดึงข้อมูลจาก Firestore
const fetchData = async () => {
  const querySnapshot = await getDocs(collection(db, "cp422101"));
  classPeriods.value = querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
};

// ฟังก์ชันเพิ่มข้อมูลใหม่
const subClassPeriod = async () => {
  await addDoc(collection(db, "cp422101"), {
    lesson: periodName.value,
    date: date.value,
    time: time.value,
  });
  await fetchData(); // อัพเดตข้อมูลหลังจากเพิ่ม
  dialog.value = false;
};

// เปิด Dialog เพื่อเพิ่มข้อมูล
const addClassPeriod = () => {
  periodName.value = '';
  date.value = '';
  time.value = '';
  dialog.value = true;
};

// ฟังก์ชันเปิด Dialog ยืนยันการลบ
const confirmDeleteClassPeriod = (id) => {
  currentDeleteId.value = id;
  deleteDialog.value = true;
};

// ฟังก์ชันลบข้อมูล
const deleteClassPeriod = async () => {
  await deleteDoc(doc(db, "cp422101", currentDeleteId.value));
  await fetchData(); // อัพเดตข้อมูลหลังจากลบ
  deleteDialog.value = false;
};

// เรียก fetchData ตอนโหลดหน้าเว็บ
fetchData();

</script>

<template>
  <div>
    <div style="display: flex; align-items: center;">
      <h1>CP422101 - Introduction to Computer Networking</h1>
    </div>
    <div style="display: flex; justify-content: space-between;">
      <p style="font-size: 15px; text-align: start;"></p>
      <p style="font-size: 18px; text-align: end;">
        <a href="/home">Classroom</a> /
      </p>
    </div>
    
    <VCard class="mt-5" :style="{ backgroundColor: '#747070', color: '#fff' }">
      <VCardText>
        <div>
          <VBtn color="primary" @click="addClassPeriod" class="mr-2">
            <v-icon left>mdi-plus</v-icon>
            Add Class Period
          </VBtn>
        </div>

        <div style="margin-block-start: 10px;">
          <VCard
            v-for="period in classPeriods"
            :key="period.id"
            :style="{ backgroundColor: '#D9D9D9', color: '#333', marginBottom: '10px' }"
          >
            <VCardText style="inline-size: 100%;">
              <div style="display: flex; align-items: center; padding: 5px; font-size: 18px;">
                <span style="flex: 1; text-align: start;">{{ period.lesson }}</span>
                <span style="flex: 1;margin-inline-end: 10px; text-align: start;">{{ period.date }}</span>
                <span style="flex: 1;margin-inline-end: 10px; text-align: start;">{{ period.time }}</span>
                <VBtn icon @click="$router.push(`/attendanceCheck/${period.id}`)" style="margin-inline-start: auto;" color="green">
                  <v-icon>mdi-page-next-outline</v-icon>
                  </VBtn>
                <VBtn icon @click="confirmDeleteClassPeriod(period.id)" style="margin-inline-start: auto;" color="red">
                  <v-icon>mdi-delete-forever</v-icon>
                </VBtn>
              </div>
            </VCardText>
          </VCard>
        </div>
      </VCardText>
    </VCard>

    <!-- Dialog สำหรับเพิ่มข้อมูล -->
    <VDialog v-model="dialog" max-width="600px"> <!-- ขยายขนาดเป็น 600px -->
      <template #title>
        <h3>Add Class Period</h3>
      </template>
      <VCard>
        <VCardText>
          <VForm>
            <VTextField 
              v-model="periodName" 
              label="Lesson Name" 
              required 
              class="mb-4" 
              style="margin-top: 20px;" 
            />
            <VTextField 
              v-model="date" 
              label="Date" 
              required 
              placeholder="e.g., 2023-09-25 or any custom format" 
            />
            <VTextField 
              v-model="time" 
              label="Time (e.g., 14:30 - 16:00)" 
              required 
              class="mt-4"
            />
            <VCardActions>
              <VBtn color="primary" @click="subClassPeriod">Submit</VBtn>
              <VBtn @click="dialog = false">Cancel</VBtn>
            </VCardActions>
          </VForm>
        </VCardText>
      </VCard>
    </VDialog>

    <!-- Dialog ยืนยันการลบ -->
    <VDialog v-model="deleteDialog" max-width="400px">
      <template #title>
        <h3>Confirm Delete</h3>
      </template>
      <VCard>
        <VCardText>
          Are you sure you want to delete this class period?
        </VCardText>
        <VCardActions>
          <VBtn color="red" @click="deleteClassPeriod">Yes</VBtn>
          <VBtn @click="deleteDialog = false">No</VBtn>
        </VCardActions>
      </VCard>
    </VDialog>
  </div>
</template>
