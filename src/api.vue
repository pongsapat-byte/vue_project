<script setup>
import { ref, computed, onMounted } from 'vue'

const title = ref('')
const className = ref('')
const teacher = ref('')
const date = ref('')
const students = ref([])
const detail = ref(null)
const loading = ref(true)
const search = ref('')

// ค้นหาจากชื่อ รหัสนักเรียน หรือเลขที่
const filteredStudents = computed(() => {
    if (!search.value) return students.value
    const keyword = search.value.toLowerCase().trim()
    return students.value.filter(s =>
        (s.full_name && s.full_name.toLowerCase().includes(keyword)) ||
        (s.student_id && String(s.student_id).toLowerCase().includes(keyword)) ||
        (s.no && String(s.no).toLowerCase().includes(keyword))
    )
})

// ตรวจสอบเพศเบื้องต้นจากคำนำหน้าชื่อ
const getGenderInfo = (name) => {
    if (!name) return { text: 'ไม่ระบุ', color: 'grey', icon: 'mdi-account' }
    if (name.startsWith('นาย') || name.startsWith('ด.ช.')) {
        return { text: 'ชาย', color: 'blue', icon: 'mdi-gender-male' }
    }
    if (name.startsWith('นาง') || name.startsWith('น.ส.') || name.startsWith('ด.ญ.')) {
        return { text: 'หญิง', color: 'pink', icon: 'mdi-gender-female' }
    }
    return { text: 'ไม่มีข้อมูล', color: 'grey', icon: 'mdi-account-question' }
}

onMounted(async () => {
    try {
        loading.value = true
        const res = await fetch('https://ntc2.technicnan.ac.th/public/nan_api/')
        if (!res.ok) throw new Error(`HTTP error: ${res.status}`)
        const data = await res.json()

        title.value = data.title || ''
        className.value = data.class || ''
        teacher.value = data.teacher || ''
        date.value = data.date || ''
        students.value = data.data || []
    } catch (error) {
        console.error('เกิดข้อผิดพลาดในการโหลดข้อมูล:', error)
    } finally {
        loading.value = false
    }
})
</script>

<template>
    <v-container class="page-shell py-6 py-md-10">
        <!-- Hero / ข้อมูลห้องเรียน -->
        <v-card class="hero-card mb-7" rounded="xl" elevation="0">
            <div class="hero-decoration hero-decoration-one"></div>
            <div class="hero-decoration hero-decoration-two"></div>
            <div class="hero-content pa-5 pa-md-8">
                <div class="d-flex flex-column flex-md-row justify-space-between align-md-center ga-6">
                    <div class="hero-main">
                        <v-chip class="hero-label mb-4" color="white" variant="flat" size="small">
                            <v-icon start icon="mdi-school-outline" size="small"></v-icon>
                            STUDENT DIRECTORY
                        </v-chip>

                        <h1 class="hero-title mb-3">
                            {{ title || 'กำลังโหลดข้อมูล...' }}
                        </h1>
                        <p class="hero-subtitle mb-5">ข้อมูลนักเรียนในที่ปรึกษา</p>

                        <div class="class-meta d-flex flex-wrap ga-2 ga-md-3">
                            <div class="meta-pill">
                                <v-icon icon="mdi-google-classroom" size="small"></v-icon>
                                <span>ชั้น <strong>{{ className || '-' }}</strong></span>
                            </div>
                            <div class="meta-pill">
                                <v-icon icon="mdi-account-tie-outline" size="small"></v-icon>
                                <span>ครูที่ปรึกษา <strong>{{ teacher || '-' }}</strong></span>
                            </div>
                            <div class="meta-pill">
                                <v-icon icon="mdi-calendar-month-outline" size="small"></v-icon>
                                <span>{{ date || '-' }}</span>
                            </div>
                        </div>
                    </div>

                    <div class="student-total">
                        <div class="total-icon">
                            <v-icon icon="mdi-account-group-outline" size="25"></v-icon>
                        </div>
                        <div class="total-caption">นักเรียนทั้งหมด</div>
                        <div class="total-number">{{ students.length }}</div>
                        <div class="total-unit">คน</div>
                    </div>
                </div>
            </div>
            <div class="hero-bottom-line"></div>
        </v-card>

        <!-- แถบเครื่องมือ -->
        <div class="toolbar d-flex flex-column flex-sm-row align-sm-center justify-space-between ga-3 mb-5">
            <div class="toolbar-heading">
                <div class="section-kicker">CLASS OVERVIEW</div>
                <h2 class="section-title">รายชื่อนักเรียน</h2>
                <p class="section-description">เลือกการ์ดนักเรียนเพื่อดูข้อมูลเพิ่มเติม</p>
            </div>
            <v-text-field
                v-model="search"
                class="search-field"
                density="comfortable"
                variant="solo"
                bg-color="white"
                rounded="xl"
                label="ค้นหาชื่อ รหัส หรือเลขที่..."
                prepend-inner-icon="mdi-magnify"
                clearable
                hide-details
            ></v-text-field>
        </div>

        <!-- สถานะโหลด -->
        <v-row v-if="loading" class="student-grid">
            <v-col cols="12" sm="6" md="4" lg="3" v-for="n in 8" :key="n">
                <v-skeleton-loader type="image, article" class="loading-card"></v-skeleton-loader>
            </v-col>
        </v-row>

        <!-- การ์ดนักเรียน -->
        <template v-else-if="filteredStudents.length > 0">
            <div class="result-count mb-3">
                <v-icon icon="mdi-account-multiple-outline" size="small"></v-icon>
                พบข้อมูล <strong>{{ filteredStudents.length }}</strong> คน
            </div>
            <v-row class="student-grid">
                <v-col
                    cols="12"
                    sm="6"
                    md="4"
                    lg="3"
                    v-for="list in filteredStudents"
                    :key="list.student_id || list.no"
                >
                    <v-card
                        class="student-card h-100"
                        rounded="xl"
                        elevation="0"
                        @click="detail = list"
                    >
                        <div class="student-photo-wrap">
                            <v-img
                                :src="list.image_url"
                                height="225"
                                cover
                                class="student-photo"
                            >
                                <template v-slot:error>
                                    <div class="photo-fallback">
                                        <v-icon icon="mdi-account-circle-outline" size="66"></v-icon>
                                        <span>ไม่มีรูปภาพ</span>
                                    </div>
                                </template>
                            </v-img>
                            <div class="photo-shade"></div>
                            <v-chip class="number-chip" color="white" variant="flat" size="small">
                                <v-icon start icon="mdi-numeric" size="small"></v-icon>
                                เลขที่ {{ list.no || '-' }}
                            </v-chip>
                            <div class="view-chip">
                                <v-icon icon="mdi-arrow-top-right" size="small"></v-icon>
                            </div>
                        </div>

                        <v-card-text class="student-info pa-4">
                            <h3 class="student-name">{{ list.full_name || 'ไม่ระบุชื่อ' }}</h3>
                            <div class="student-code">
                                <v-icon icon="mdi-card-account-details-outline" size="16"></v-icon>
                                <span>รหัส {{ list.student_id || '-' }}</span>
                            </div>
                            <v-divider class="my-4"></v-divider>
                            <div class="d-flex align-center justify-space-between">
                                <span class="info-caption">ข้อมูลส่วนตัว</span>
                                <v-chip
                                    :color="getGenderInfo(list.full_name).color"
                                    variant="tonal"
                                    size="small"
                                    class="gender-chip"
                                >
                                    <v-icon start :icon="getGenderInfo(list.full_name).icon" size="small"></v-icon>
                                    {{ getGenderInfo(list.full_name).text }}
                                </v-chip>
                            </div>
                        </v-card-text>
                    </v-card>
                </v-col>
            </v-row>
        </template>

        <!-- ไม่พบข้อมูล -->
        <v-card v-else class="empty-state pa-8 pa-md-12 text-center" rounded="xl" elevation="0">
            <div class="empty-icon mx-auto mb-4">
                <v-icon icon="mdi-account-search-outline" size="42"></v-icon>
            </div>
            <h3 class="text-h6 font-weight-bold mb-2">ไม่พบข้อมูลนักเรียน</h3>
            <p class="text-body-2 text-medium-emphasis mb-5">ลองตรวจสอบคำค้นหา หรือลองค้นหาด้วยชื่อและรหัสนักเรียนอีกครั้ง</p>
            <v-btn color="primary" variant="tonal" rounded="lg" @click="search = ''">
                <v-icon start icon="mdi-refresh"></v-icon>
                ล้างคำค้นหา
            </v-btn>
        </v-card>

        <!-- หน้าต่างรายละเอียด -->
        <v-dialog v-model="detail" max-width="460">
            <v-card v-if="detail" class="detail-dialog" rounded="xl" elevation="12">
                <div class="dialog-header pa-5">
                    <div class="d-flex align-center justify-space-between">
                        <div>
                            <div class="dialog-kicker">STUDENT PROFILE</div>
                            <div class="text-h6 font-weight-bold text-white">ข้อมูลประจำตัวนักเรียน</div>
                        </div>
                        <v-btn
                            icon="mdi-close"
                            variant="tonal"
                            color="white"
                            size="small"
                            aria-label="ปิดหน้าต่าง"
                            @click="detail = null"
                        ></v-btn>
                    </div>
                </div>

                <v-card-text class="pa-6 text-center">
                    <v-avatar size="154" class="profile-avatar mb-4">
                        <v-img :src="detail.image_url" cover>
                            <template v-slot:error>
                                <div class="profile-fallback">
                                    <v-icon icon="mdi-account" size="64"></v-icon>
                                </div>
                            </template>
                        </v-img>
                    </v-avatar>

                    <div class="mb-3">
                        <v-chip color="primary" variant="tonal" size="small" class="font-weight-bold">
                            เลขที่ {{ detail.no || '-' }}
                        </v-chip>
                    </div>
                    <h2 class="profile-name mb-1">{{ detail.full_name || 'ไม่ระบุชื่อ' }}</h2>
                    <p class="profile-code mb-5">รหัสนักเรียน: {{ detail.student_id || '-' }}</p>

                    <v-list class="profile-list text-left pa-2" rounded="lg" bg-color="grey-lighten-4" density="comfortable">
                        <v-list-item>
                            <template v-slot:prepend>
                                <v-avatar color="primary" variant="tonal" size="36">
                                    <v-icon icon="mdi-school-outline" size="19"></v-icon>
                                </v-avatar>
                            </template>
                            <v-list-item-title class="text-body-2 text-medium-emphasis">ระดับชั้น</v-list-item-title>
                            <v-list-item-subtitle class="font-weight-bold text-grey-darken-4">{{ className || '-' }}</v-list-item-subtitle>
                        </v-list-item>
                        <v-list-item>
                            <template v-slot:prepend>
                                <v-avatar :color="getGenderInfo(detail.full_name).color" variant="tonal" size="36">
                                    <v-icon :icon="getGenderInfo(detail.full_name).icon" size="19"></v-icon>
                                </v-avatar>
                            </template>
                            <v-list-item-title class="text-body-2 text-medium-emphasis">เพศ</v-list-item-title>
                            <v-list-item-subtitle class="font-weight-bold text-grey-darken-4">{{ getGenderInfo(detail.full_name).text }}</v-list-item-subtitle>
                        </v-list-item>
                    </v-list>
                </v-card-text>

                <v-card-actions class="px-6 pb-6 pt-0">
                    <v-btn color="primary" variant="flat" block size="large" rounded="lg" class="font-weight-bold" @click="detail = null">
                        ปิดหน้าต่าง
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <div class="page-footer mt-8">
            <span>STUDENT DIRECTORY</span>
            <span class="footer-dot"></span>
            <span>ระบบแสดงข้อมูลนักเรียน</span>
        </div>
    </v-container>
</template>

<style scoped>
.page-shell {
    max-width: 1380px;
    min-height: 100vh;
    position: relative;
}

:global(body) {
    background: #f5f7fc;
}

.hero-card {
    position: relative;
    overflow: hidden;
    color: #fff;
    background: linear-gradient(120deg, #152b68 0%, #3156ba 52%, #7654c8 100%);
    box-shadow: 0 18px 45px rgba(43, 69, 145, 0.18) !important;
}

.hero-content {
    position: relative;
    z-index: 2;
}

.hero-decoration {
    position: absolute;
    border: 1px solid rgba(255, 255, 255, 0.13);
    border-radius: 50%;
    pointer-events: none;
}

.hero-decoration-one {
    width: 360px;
    height: 360px;
    top: -190px;
    right: 16%;
    box-shadow: 0 0 0 35px rgba(255, 255, 255, 0.035), 0 0 0 75px rgba(255, 255, 255, 0.025);
}

.hero-decoration-two {
    width: 240px;
    height: 240px;
    right: -75px;
    bottom: -150px;
    background: rgba(255, 255, 255, 0.055);
}

.hero-label {
    color: #29449a !important;
    font-weight: 800;
    letter-spacing: 0.08em;
}

.hero-title {
    max-width: 760px;
    color: #fff;
    font-size: clamp(1.55rem, 3vw, 2.45rem);
    font-weight: 800;
    line-height: 1.25;
    letter-spacing: -0.025em;
}

.hero-subtitle {
    color: rgba(255, 255, 255, 0.78);
    font-size: 1rem;
    letter-spacing: 0.02em;
}

.class-meta {
    color: rgba(255, 255, 255, 0.94);
}

.meta-pill {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    padding: 9px 13px;
    border: 1px solid rgba(255, 255, 255, 0.19);
    border-radius: 12px;
    background: rgba(255, 255, 255, 0.105);
    backdrop-filter: blur(8px);
    font-size: 0.85rem;
}

.meta-pill strong {
    color: #fff;
    font-weight: 700;
}

.student-total {
    flex: 0 0 172px;
    min-width: 172px;
    padding: 20px 18px;
    border: 1px solid rgba(255, 255, 255, 0.23);
    border-radius: 20px;
    background: rgba(255, 255, 255, 0.13);
    text-align: center;
    backdrop-filter: blur(12px);
    box-shadow: 0 12px 30px rgba(15, 25, 75, 0.12);
}

.total-icon {
    display: grid;
    place-items: center;
    width: 43px;
    height: 43px;
    margin: 0 auto 10px;
    border-radius: 14px;
    color: #fff;
    background: rgba(255, 255, 255, 0.2);
}

.total-caption,
.total-unit {
    color: rgba(255, 255, 255, 0.82);
    font-size: 0.82rem;
}

.total-number {
    margin: 1px 0;
    color: #fff;
    font-size: 2.8rem;
    font-weight: 800;
    line-height: 1.15;
    font-variant-numeric: tabular-nums;
}

.hero-bottom-line {
    position: absolute;
    right: 0;
    bottom: 0;
    left: 0;
    height: 4px;
    background: linear-gradient(90deg, #68dcff, #a9a0ff, #f5a9dc);
}

.toolbar {
    padding: 2px 2px 0;
}

.section-kicker {
    color: #6475b1;
    font-size: 0.69rem;
    font-weight: 800;
    letter-spacing: 0.14em;
}

.section-title {
    margin-top: 3px;
    color: #202b49;
    font-size: 1.45rem;
    font-weight: 800;
}

.section-description {
    margin-top: 3px;
    color: #7c8499;
    font-size: 0.87rem;
}

.search-field {
    width: min(100%, 390px);
}

:deep(.search-field .v-field) {
    border: 1px solid #e7eaf3;
    box-shadow: 0 5px 18px rgba(30, 49, 100, 0.045) !important;
}

:deep(.search-field .v-field__input) {
    font-size: 0.9rem;
}

.result-count {
    display: flex;
    align-items: center;
    gap: 7px;
    color: #7a839b;
    font-size: 0.86rem;
}

.result-count strong {
    color: #354fa8;
}

.student-grid {
    row-gap: 4px;
}

.student-card {
    overflow: hidden;
    height: 100%;
    cursor: pointer;
    border: 1px solid #e9edf6;
    background: #fff;
    box-shadow: 0 5px 18px rgba(34, 52, 100, 0.055) !important;
    transition: transform 220ms ease, box-shadow 220ms ease, border-color 220ms ease;
}

.student-card:hover {
    transform: translateY(-6px);
    border-color: #b9c7f4;
    box-shadow: 0 16px 30px rgba(53, 79, 168, 0.14) !important;
}

.student-photo-wrap {
    position: relative;
    overflow: hidden;
    background: linear-gradient(135deg, #e7edff, #f1eaff);
}

.student-photo {
    transition: transform 350ms ease;
}

.student-card:hover .student-photo {
    transform: scale(1.045);
}

.photo-shade {
    position: absolute;
    inset: 45% 0 0;
    pointer-events: none;
    background: linear-gradient(transparent, rgba(19, 28, 61, 0.22));
}

.number-chip {
    position: absolute;
    top: 13px;
    left: 13px;
    color: #344da4 !important;
    font-weight: 800;
    box-shadow: 0 4px 12px rgba(20, 30, 70, 0.12);
}

.view-chip {
    position: absolute;
    right: 13px;
    bottom: 13px;
    display: grid;
    place-items: center;
    width: 31px;
    height: 31px;
    border: 1px solid rgba(255, 255, 255, 0.38);
    border-radius: 10px;
    color: #fff;
    background: rgba(25, 35, 75, 0.35);
    backdrop-filter: blur(6px);
    opacity: 0;
    transform: translateY(4px);
    transition: all 180ms ease;
}

.student-card:hover .view-chip {
    opacity: 1;
    transform: translateY(0);
}

.photo-fallback,
.profile-fallback {
    display: flex;
    height: 100%;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    gap: 7px;
    color: #8b98bd;
    font-size: 0.8rem;
    background: linear-gradient(135deg, #edf1ff, #f5f0ff);
}

.student-info {
    min-height: 150px;
}

.student-name {
    overflow: hidden;
    color: #263252;
    font-size: 1rem;
    font-weight: 800;
    line-height: 1.5;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.student-code {
    display: flex;
    align-items: center;
    gap: 6px;
    margin-top: 6px;
    color: #818aa1;
    font-size: 0.83rem;
}

.info-caption {
    color: #9aa1b2;
    font-size: 0.76rem;
}

.gender-chip {
    font-weight: 700;
}

.loading-card {
    overflow: hidden;
    border: 1px solid #edf0f7;
    border-radius: 18px;
}

.empty-state {
    border: 1px dashed #d9dfef;
    background: rgba(255, 255, 255, 0.8);
}

.empty-icon {
    display: grid;
    place-items: center;
    width: 82px;
    height: 82px;
    border-radius: 25px;
    color: #6579c5;
    background: #edf1ff;
}

.detail-dialog {
    overflow: hidden;
    background: #fff;
}

.dialog-header {
    color: #fff;
    background: linear-gradient(120deg, #1c347c, #536bc5 70%, #7a5bc4);
}

.dialog-kicker {
    margin-bottom: 4px;
    color: rgba(255, 255, 255, 0.7);
    font-size: 0.65rem;
    font-weight: 800;
    letter-spacing: 0.15em;
}

.profile-avatar {
    border: 5px solid #fff;
    background: #edf1ff;
    box-shadow: 0 8px 25px rgba(39, 57, 120, 0.18);
}

.profile-name {
    color: #263252;
    font-size: 1.2rem;
    font-weight: 800;
}

.profile-code {
    color: #6475b1;
    font-size: 0.9rem;
    font-weight: 600;
}

.profile-list {
    border: 1px solid #edf0f7;
}

.page-footer {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    color: #a0a7b8;
    font-size: 0.7rem;
    font-weight: 700;
    letter-spacing: 0.08em;
}

.footer-dot {
    width: 4px;
    height: 4px;
    border-radius: 50%;
    background: #aeb8d8;
}

@media (max-width: 600px) {
    .hero-title {
        font-size: 1.55rem;
    }

    .student-total {
        display: grid;
        grid-template-columns: auto 1fr auto auto;
        align-items: center;
        gap: 9px;
        min-width: 0;
        padding: 12px 15px;
        text-align: left;
    }

    .total-icon {
        width: 38px;
        height: 38px;
        margin: 0;
    }

    .total-caption {
        font-size: 0.78rem;
    }

    .total-number {
        font-size: 1.6rem;
    }

    .total-unit {
        align-self: end;
        padding-bottom: 3px;
    }

    .search-field {
        width: 100%;
    }

    .view-chip {
        opacity: 1;
        transform: none;
    }
}

@media (prefers-reduced-motion: reduce) {
    *,
    *::before,
    *::after {
        scroll-behavior: auto !important;
        transition-duration: 0.01ms !important;
        animation-duration: 0.01ms !important;
    }
}
</style>
