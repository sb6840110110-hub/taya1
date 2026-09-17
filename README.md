<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ClassCraft Studio - ระบบห้องเรียนพร้อมระบบลงทะเบียน</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Sarabun', sans-serif; }
        .glass {
            background: rgba(255, 255, 255, 0.9);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.5);
        }
    </style>
</head>
<body class="bg-gradient-to-br from-indigo-100 via-purple-50 to-pink-100 min-h-screen text-gray-800">

    <!-- Navbar -->
    <nav class="glass sticky top-0 z-50 px-6 py-4 flex justify-between items-center shadow-sm">
        <div class="flex items-center space-x-3">
            <div class="bg-indigo-600 text-white p-2 rounded-xl shadow-md font-bold text-lg">✨ CS</div>
            <span class="text-xl font-bold bg-gradient-to-r from-indigo-600 to-pink-600 bg-clip-text text-transparent">ClassCraft Studio</span>
        </div>
        
        <div class="flex items-center space-x-3">
            <div class="relative">
                <button onclick="openNotificationModal()" class="relative bg-white hover:bg-gray-100 p-2.5 rounded-xl border border-gray-200 shadow-sm transition text-gray-600" title="การแจ้งเตือน">
                    🔔
                    <span id="globalNotifBadge" class="absolute -top-1 -right-1 bg-red-500 text-white text-[10px] font-bold px-1.5 py-0.2 rounded-full hidden shadow">0</span>
                </button>
            </div>

            <div id="currentLoggedUser" class="hidden text-xs bg-pink-100 text-pink-700 px-3 py-1.5 rounded-xl font-bold border border-pink-200">
                👤 นร: <span id="userNameDisplay">-</span> (<span id="userIdDisplay">-</span>)
            </div>

            <!-- ปุ่มสร้างห้องเรียน (เฉพาะครู) -->
            <button id="createClassBtn" onclick="openCreateModal()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-xl shadow-lg transition text-sm font-medium">
                + สร้างห้องเรียนใหม่
            </button>

            <!-- ปุ่มลงทะเบียนเข้าห้องเรียน (เฉพาะนักเรียน) -->
            <button id="joinClassBtn" onclick="openJoinModal()" class="bg-pink-600 hover:bg-pink-700 text-white px-4 py-2 rounded-xl shadow-lg transition text-sm font-medium hidden">
                📝 ลงทะเบียนเข้าห้องเรียน
            </button>

            <button id="simulateStudentBtn" onclick="switchToStudentPrompt()" class="bg-pink-100 hover:bg-pink-200 text-pink-700 px-3 py-2 rounded-xl text-xs font-bold transition border border-pink-200">
                👁️ ไปมุมมองนักเรียน
            </button>

            <button id="openTeacherLoginBtn" onclick="openTeacherLoginModal()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-3 py-2 rounded-xl shadow transition text-xs font-medium hidden">
                👩‍🏫 เข้าสู่โหมดครู
            </button>

            <button id="setupTeacherPassBtn" onclick="openSetupPasswordModal()" class="bg-gray-200 hover:bg-gray-300 text-gray-700 px-2.5 py-2 rounded-xl text-xs font-semibold transition" title="ตั้งค่ารหัสผ่านโหมดครู">
                ⚙️
            </button>

            <div id="roleAvatar" class="w-10 h-10 rounded-full bg-indigo-200 flex items-center justify-center font-bold text-indigo-700 shadow-inner">คร</div>
        </div>
    </nav>

    <!-- Main Container -->
    <main class="max-w-6xl mx-auto p-6">
        <header class="mb-8">
            <h1 id="mainTitle" class="text-3xl font-bold text-gray-900">ห้องเรียนของฉัน (มุมมองคุณครูผู้ควบคุม)</h1>
            <p id="mainSubtitle" class="text-gray-500 mt-1">จัดการห้องเรียน สร้างงาน ประกาศข่าวสาร และตรวจสอบข้อมูลการลงทะเบียนของนักเรียน</p>
        </header>

        <!-- Empty State Alert -->
        <div id="empty-state" class="text-center py-16 bg-white/50 backdrop-blur-sm border-2 border-dashed border-gray-300 rounded-3xl p-8">
            <div class="text-5xl mb-3">🏫</div>
            <h3 class="text-lg font-bold text-gray-700 mb-1">ยังไม่มีห้องเรียนในระบบ</h3>
            <p id="emptyStateDesc" class="text-xs text-gray-500 mb-4">เริ่มต้นใช้งานโดยการกดปุ่มสร้างห้องเรียนใหม่ด้านบน</p>
            
            <button id="emptyCreateBtn" onclick="openCreateModal()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-5 py-2 rounded-xl text-xs font-semibold shadow transition">
                + สร้างห้องเรียนแรก
            </button>
            <button id="emptyJoinBtn" onclick="openJoinModal()" class="bg-pink-600 hover:bg-pink-700 text-white px-5 py-2 rounded-xl text-xs font-semibold shadow transition hidden">
                📝 ลงทะเบียนเข้าห้องเรียน
            </button>
        </div>

        <!-- Classroom Grid -->
        <div id="classroom-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6"></div>
    </main>

    <!-- Modal: หน้าภายในห้องเรียน -->
    <div id="roomModal" class="fixed inset-0 bg-black/40 backdrop-blur-sm hidden z-50 flex items-center justify-center p-4">
        <div class="glass bg-white rounded-3xl p-6 w-full max-w-4xl shadow-2xl max-h-[90vh] flex flex-col">
            <div class="flex justify-between items-center border-b pb-4 mb-4">
                <div>
                    <h2 id="roomTitle" class="text-2xl font-bold text-gray-800">ชื่อห้องเรียน</h2>
                    <p id="roomCode" class="text-xs text-indigo-600 mt-0.5">รหัสห้องเรียน: -</p>
                </div>
                <button onclick="closeClassroom()" class="text-gray-400 hover:text-gray-600 font-bold text-xl">✕</button>
            </div>

            <!-- เมนูสลับแท็บ -->
            <div class="flex space-x-2 border-b border-gray-200 pb-3 mb-4">
                <button onclick="switchRoomTab('stream')" id="tabStreamBtn" class="relative px-4 py-2 rounded-xl font-bold text-sm bg-indigo-600 text-white shadow transition">
                    💬 กระดานประกาศ (Stream)
                </button>
                <button onclick="switchRoomTab('assignments')" id="tabAssignmentsBtn" class="relative px-4 py-2 rounded-xl font-bold text-sm bg-gray-100 text-gray-600 hover:bg-gray-200 transition">
                    📚 งานที่มอบหมาย (Assignments)
                </button>
            </div>

            <!-- ส่วนสำหรับครูควบคุม -->
            <div id="teacherControlBox" class="bg-indigo-50/70 p-4 rounded-2xl border border-indigo-100 mb-4 flex flex-col md:flex-row justify-between items-center gap-3">
                <div>
                    <h3 class="font-bold text-indigo-900 text-sm">👩‍🏫 แผงควบคุมคุณครู</h3>
                    <p class="text-xs text-indigo-700">ตรวจสอบทะเบียนรายชื่อนักเรียน หรือสร้างคำสั่งงานใหม่</p>
                </div>
                <div class="flex space-x-2">
                    <button onclick="openStudentListModal()" class="bg-purple-600 hover:bg-purple-700 text-white px-3.5 py-2 rounded-xl text-sm font-medium shadow transition">
                        👥 ทะเบียนนักเรียน (<span id="modalStudentCount">0</span>)
                    </button>
                    <button onclick="openAssignmentModal()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-3.5 py-2 rounded-xl text-sm font-medium shadow transition">
                        + สร้างคำสั่งงานใหม่
                    </button>
                </div>
            </div>

            <!-- CONTAINER 1: แท็บกระดานประกาศ (Stream) -->
            <div id="roomTabStream" class="overflow-y-auto flex-1 pr-2 space-y-4">
                <div class="bg-white p-4 rounded-2xl border border-indigo-100 shadow-sm">
                    <h4 class="font-bold text-xs text-indigo-900 mb-2">📢 ประกาศหรือพูดคุยอะไรกับชั้นเรียนของคุณ...</h4>
                    <textarea id="streamPostInput" rows="2" class="w-full p-3 text-sm border border-gray-200 rounded-xl focus:ring-2 focus:ring-indigo-500 outline-none mb-2" placeholder="เขียนข้อความหรือประกาศที่นี่..."></textarea>
                    <div class="flex justify-end">
                        <button onclick="postToStream()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-1.5 rounded-xl text-xs font-medium shadow transition">
                            โพสต์ข้อความ
                        </button>
                    </div>
                </div>
                <div id="streamPostsContainer" class="space-y-3"></div>
            </div>

            <!-- CONTAINER 2: แท็บรายการงาน (Assignments) -->
            <div id="roomTabAssignments" class="overflow-y-auto flex-1 pr-2 space-y-4 hidden">
                <div id="assignment-list" class="space-y-4"></div>
            </div>
        </div>
    </div>

    <!-- Modal: ลงทะเบียนเข้าห้องเรียน (Registration Modal) -->
    <div id="joinModal" class="fixed inset-0 bg-black/50 backdrop-blur-sm hidden z-50 flex items-center justify-center p-4">
        <div class="glass bg-white rounded-3xl p-6 w-full max-w-md shadow-2xl border-2 border-pink-100">
            <div class="text-center mb-4">
                <span class="bg-pink-100 text-pink-700 text-xs font-bold px-3 py-1 rounded-full">📝 ระบบลงทะเบียนนักเรียน</span>
                <h2 class="text-xl font-bold mt-2 text-gray-800">กรอกข้อมูลเพื่อลงทะเบียนเข้าห้องเรียน</h2>
                <p class="text-xs text-gray-500 mt-1">นักเรียนจะต้องลงทะเบียนและระบุรหัสประจำตัวก่อนเข้าห้องเรียนทุกครั้ง</p>
            </div>
            <form onsubmit="registerAndJoinClassroom(event)">
                <label class="block text-xs font-bold text-gray-700 mb-1">รหัสประจำตัวนักเรียน (Student ID)</label>
                <input type="text" id="studentIdInput" required class="w-full px-4 py-2 border rounded-xl mb-3 focus:ring-2 focus:ring-pink-500 outline-none text-sm font-semibold" placeholder="เช่น 68102">
                
                <label class="block text-xs font-bold text-gray-700 mb-1">ชื่อ - นามสกุล นักเรียน</label>
                <input type="text" id="studentNameInput" required class="w-full px-4 py-2 border rounded-xl mb-3 focus:ring-2 focus:ring-pink-500 outline-none text-sm" placeholder="เช่น เด็กชายรักเรียน เพียรศึกษา">
                
                <label class="block text-xs font-bold text-gray-700 mb-1">รหัสห้องเรียน (Class Code)</label>
                <input type="text" id="joinCodeInput" required class="w-full px-4 py-2 border rounded-xl mb-6 focus:ring-2 focus:ring-pink-500 outline-none uppercase tracking-wider font-bold text-center text-lg text-pink-600" placeholder="เช่น CS101">
                
                <div class="flex justify-end space-x-3">
                    <button type="button" onclick="closeJoinModal()" class="px-4 py-2 text-gray-600 hover:bg-gray-100 rounded-xl text-sm">ยกเลิก</button>
                    <button type="submit" class="px-4 py-2 bg-pink-600 hover:bg-pink-700 text-white rounded-xl text-sm font-medium shadow-md">ลงทะเบียนและเข้าห้องเรียน</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Modal: ตรวจงาน ให้คะแนน และคอมเมนต์ส่วนตัว (สำหรับครู) -->
    <div id="checkSubmissionsModal" class="fixed inset-0 bg-black/40 backdrop-blur-sm hidden z-50 flex items-center justify-center p-4">
        <div class="glass bg-white rounded-3xl p-6 w-full max-w-xl shadow-2xl">
            <h2 class="text-xl font-bold mb-1 text-gray-800">📊 ตรวจงานและให้คะแนนนักเรียน</h2>
            <p id="checkAssignTitle" class="text-xs text-indigo-600 font-semibold mb-4">หัวข้องาน: -</p>
            
            <div class="flex justify-between items-center mb-2 px-1 text-xs text-gray-500 font-bold">
                <span>รายชื่อนักเรียนที่ลงทะเบียน</span>
                <span>สถานะและตรวจงาน</span>
            </div>

            <div id="submissionsListContainer" class="bg-gray-50 border border-gray-200 rounded-2xl p-3 max-h-72 overflow-y-auto space-y-2 mb-4"></div>

            <div class="flex justify-end">
                <button type="button" onclick="closeCheckSubmissionsModal()" class="px-4 py-2 bg-gray-200 hover:bg-gray-300 text-gray-700 rounded-xl text-sm font-medium">ปิด</button>
            </div>
        </div>
    </div>

    <!-- Modal: ให้คะแนนและคอมเมนต์รายบุคคล -->
    <div id="gradeStudentModal" class="fixed inset-0 bg-black/40 backdrop-blur-sm hidden z-50 flex items-center justify-center p-4">
        <div class="glass bg-white rounded-3xl p-6 w-full max-w-lg shadow-2xl">
            <h2 class="text-xl font-bold mb-1 text-gray-800">✍️ ให้คะแนนและข้อเสนอแนะ</h2>
            <p id="gradeModalStudentName" class="text-xs text-indigo-600 font-semibold mb-3">นักเรียน: -</p>
            
            <div class="bg-indigo-50/60 p-4 rounded-2xl border border-indigo-100 mb-4 space-y-3">
                <div class="flex justify-between items-center text-xs">
                    <span>📁 ไฟล์งาน: <strong id="gradeModalFileName" class="text-indigo-900">-</strong></span>
                    <button onclick="openPreviewWork()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-2.5 py-1 rounded-lg font-medium shadow">🔍 เปิดพรีวิวไฟล์</button>
                </div>
                <div>
                    <span class="text-xs font-bold text-gray-500 block mb-1">🕒 เวลาส่ง: <span id="gradeModalTime" class="font-normal text-gray-700">-</span></span>
                </div>

                <hr class="border-indigo-100">

                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-bold text-gray-700 mb-1">ให้คะแนน (เช่น 10/10)</label>
                        <input type="text" id="inputGradeScore" class="w-full px-3 py-2 text-sm bg-white border rounded-xl outline-none font-bold text-indigo-600" placeholder="เช่น 9.5">
                    </div>
                </div>

                <div>
                    <label class="block text-xs font-bold text-gray-700 mb-1">💬 คอมเมนต์ส่วนตัวถึงนักเรียน</label>
                    <textarea id="inputPrivateComment" rows="2" class="w-full p-2.5 text-sm bg-white border rounded-xl outline-none" placeholder="พิมพ์ข้อความแนะนำตินชมงานนี้..."></textarea>
                </div>
            </div>

            <div class="flex justify-end space-x-2">
                <button type="button" onclick="saveGradeAndFeedback()" class="px-4 py-2 bg-green-600 hover:bg-green-700 text-white rounded-xl text-sm font-medium shadow">💾 บันทึกคะแนนและส่งคอมเมนต์</button>
                <button type="button" onclick="closeGradeStudentModal()" class="px-4 py-2 bg-gray-200 hover:bg-gray-300 text-gray-700 rounded-xl text-sm font-medium">ปิด</button>
            </div>
        </div>
    </div>

    <!-- Modal: พรีวิวไฟล์งานออนไลน์ -->
    <div id="filePreviewModal" class="fixed inset-0 bg-black/60 backdrop-blur-sm hidden z-50 flex items-center justify-center p-4">
        <div class="glass bg-white rounded-3xl p-6 w-full max-w-2xl shadow-2xl flex flex-col max-h-[85vh]">
            <div class="flex justify-between items-center border-b pb-3 mb-3">
                <h3 class="font-bold text-gray-800 text-base">🔍 พรีวิวไฟล์งาน: <span id="previewFileNameTitle" class="text-indigo-600">-</span></h3>
                <button onclick="closeFilePreviewModal()" class="text-gray-400 hover:text-gray-600 font-bold text-lg">✕</button>
            </div>
            <div id="previewContentContainer" class="flex-1 bg-gray-100 rounded-2xl flex items-center justify-center overflow-auto p-4 border border-gray-200 min-h-[300px]">
                <p class="text-sm text-gray-500">กำลังโหลดตัวอย่างไฟล์...</p>
            </div>
            <div class="flex justify-end mt-4">
                <button onclick="closeFilePreviewModal()" class="px-4 py-2 bg-gray-200 hover:bg-gray-300 text-gray-700 rounded-xl text-sm font-medium">ปิดหน้าต่าง</button>
            </div>
        </div>
    </div>

    <!-- Modal: ศูนย์รวมการแจ้งเตือน -->
    <div id="notificationModal" class="fixed inset-0 bg-black/40 backdrop-blur-sm hidden z-50 flex items-center justify-center p-4">
        <div class="glass bg-white rounded-3xl p-6 w-full max-w-md shadow-2xl">
            <div class="flex justify-between items-center border-b pb-3 mb-3">
                <h2 class="text-lg font-bold text-gray-800">🔔 ประวัติการแจ้งเตือนทั้งหมด</h2>
                <button onclick="closeNotificationModal()" class="text-gray-400 hover:text-gray-600 font-bold">✕</button>
            </div>
            <div id="notificationListContainer" class="space-y-2 max-h-72 overflow-y-auto pr-1">
                <p class="text-xs text-gray-400 text-center py-6">ยังไม่มีการแจ้งเตือนใหม่ในขณะนี้</p>
            </div>
            <div class="flex justify-end mt-4">
                <button onclick="closeNotificationModal()" class="px-4 py-2 bg-gray-200 hover:bg-gray-300 text-gray-700 rounded-xl text-sm font-medium">ปิด</button>
            </div>
        </div>
    </div>

    <!-- Modal: ดูทะเบียนรายชื่อนักเรียน (สำหรับครู) -->
    <div id="studentListModal" class="fixed inset-0 bg-black/40 backdrop-blur-sm hidden z-50 flex items-center justify-center p-4">
        <div class="glass bg-white rounded-3xl p-6 w-full max-w-md shadow-2xl">
            <h2 class="text-xl font-bold mb-1 text-gray-800">👥 ทะเบียนรายชื่อนักเรียน</h2>
            <p class="text-xs text-gray-500 mb-4">รายชื่อและรหัสประจำตัวเด็กๆ ที่ลงทะเบียนเข้าเรียนในห้องนี้</p>
            <div id="studentListContainer" class="bg-gray-50 border border-gray-200 rounded-2xl p-3 max-h-60 overflow-y-auto space-y-2 mb-4"></div>
            <div class="flex justify-end">
                <button type="button" onclick="closeStudentListModal()" class="px-4 py-2 bg-gray-200 hover:bg-gray-300 text-gray-700 rounded-xl text-sm font-medium">ปิด</button>
            </div>
        </div>
    </div>

    <!-- Modal: สร้างห้องเรียนใหม่ (สำหรับครูเท่านั้น) -->
    <div id="createModal" class="fixed inset-0 bg-black/40 backdrop-blur-sm hidden z-50 flex items-center justify-center p-4">
        <div class="glass bg-white rounded-3xl p-6 w-full max-w-md shadow-2xl">
            <h2 class="text-xl font-bold mb-4 text-gray-800">สร้างห้องเรียนใหม่</h2>
            <form onsubmit="addClassroom(event)">
                <label class="block text-sm font-medium text-gray-700 mb-1">ชื่อวิชา / ห้องเรียน</label>
                <input type="text" id="className" required class="w-full px-4 py-2 border rounded-xl mb-4 focus:ring-2 focus:ring-indigo-500 outline-none" placeholder="เช่น วิชาวิทยาศาสตร์ ม.1">
                <label class="block text-sm font-medium text-gray-700 mb-1">ชื่อคุณครูผู้สอน</label>
                <input type="text" id="teacherName" required class="w-full px-4 py-2 border rounded-xl mb-6 focus:ring-2 focus:ring-indigo-500 outline-none" placeholder="เช่น ครูดารารัตน์">
                <div class="flex justify-end space-x-3">
                    <button type="button" onclick="closeCreateModal()" class="px-4 py-2 text-gray-600 hover:bg-gray-100 rounded-xl text-sm">ยกเลิก</button>
                    <button type="submit" class="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl text-sm font-medium shadow-md">สร้างห้องเรียน</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Modal: จำลองมุมมองนักเรียน -->
    <div id="studentPromptModal" class="fixed inset-0 bg-black/40 backdrop-blur-sm hidden z-50 flex items-center justify-center p-4">
        <div class="glass bg-white rounded-3xl p-6 w-full max-w-sm shadow-2xl">
            <h2 class="text-xl font-bold mb-2 text-gray-800">👁️ สลับมุมมองนักเรียน</h2>
            <p class="text-xs text-gray-500 mb-4">กรอกข้อมูลนักเรียนเพื่อจำลองการใช้งาน</p>
            <form onsubmit="confirmStudentSimulation(event)">
                <label class="block text-sm font-medium text-gray-700 mb-1">รหัสประจำตัวนักเรียน</label>
                <input type="text" id="simulateIdInput" required class="w-full px-4 py-2 border rounded-xl mb-3 focus:ring-2 focus:ring-pink-500 outline-none text-sm" placeholder="เช่น 68101">
                <label class="block text-sm font-medium text-gray-700 mb-1">ชื่อนักเรียน</label>
                <input type="text" id="simulateNameInput" required class="w-full px-4 py-2 border rounded-xl mb-6 focus:ring-2 focus:ring-pink-500 outline-none text-sm" placeholder="เช่น ด.ช. อภิสิทธิ์ เรียนดี">
                <div class="flex justify-end space-x-3">
                    <button type="button" onclick="closeStudentPromptModal()" class="px-4 py-2 text-gray-600 hover:bg-gray-100 rounded-xl text-sm">ยกเลิก</button>
                    <button type="submit" class="px-4 py-2 bg-pink-600 hover:bg-pink-700 text-white rounded-xl text-sm font-medium shadow-md">เข้าสู่โหมดนักเรียน</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Modal: ใส่รหัสผ่านเข้าโหมดครู -->
    <div id="teacherLoginModal" class="fixed inset-0 bg-black/40 backdrop-blur-sm hidden z-50 flex items-center justify-center p-4">
        <div class="glass bg-white rounded-3xl p-6 w-full max-w-sm shadow-2xl">
            <h2 class="text-xl font-bold mb-2 text-gray-800">🔐 เข้าสู่โหมดคุณครู</h2>
            <p class="text-xs text-gray-500 mb-4">กรอกรหัสผ่านเพื่อควบคุมระบบ</p>
            <form onsubmit="verifyTeacherPassword(event)">
                <label class="block text-sm font-medium text-gray-700 mb-1">รหัสผ่านโหมดครู</label>
                <input type="password" id="teacherPassInput" required class="w-full px-4 py-2 border rounded-xl mb-6 focus:ring-2 focus:ring-indigo-500 outline-none text-center font-bold tracking-widest text-lg" placeholder="••••••••">
                <div class="flex justify-end space-x-3">
                    <button type="button" onclick="closeTeacherLoginModal()" class="px-4 py-2 text-gray-600 hover:bg-gray-100 rounded-xl text-sm">ยกเลิก</button>
                    <button type="submit" class="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl text-sm font-medium shadow-md">ยืนยัน</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Modal: ตั้งรหัสผ่านโหมดครู -->
    <div id="setupPasswordModal" class="fixed inset-0 bg-black/40 backdrop-blur-sm hidden z-50 flex items-center justify-center p-4">
        <div class="glass bg-white rounded-3xl p-6 w-full max-w-sm shadow-2xl">
            <h2 class="text-xl font-bold mb-2 text-gray-800">⚙️ กำหนดรหัสผ่านโหมดครู</h2>
            <form onsubmit="saveTeacherPassword(event)">
                <label class="block text-sm font-medium text-gray-700 mb-1">รหัสผ่านใหม่</label>
                <input type="password" id="newTeacherPass" required class="w-full px-4 py-2 border rounded-xl mb-4 focus:ring-2 focus:ring-indigo-500 outline-none" placeholder="รหัสผ่านใหม่">
                <div class="flex justify-end space-x-3">
                    <button type="button" onclick="closeSetupPasswordModal()" class="px-4 py-2 text-gray-600 hover:bg-gray-100 rounded-xl text-sm">ยกเลิก</button>
                    <button type="submit" class="px-4 py-2 bg-green-600 hover:bg-green-700 text-white rounded-xl text-sm font-medium shadow-md">บันทึก</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Modal: สร้างคำสั่งงานใหม่ -->
    <div id="assignmentModal" class="fixed inset-0 bg-black/40 backdrop-blur-sm hidden z-50 flex items-center justify-center p-4">
        <div class="glass bg-white rounded-3xl p-6 w-full max-w-md shadow-2xl">
            <h2 class="text-xl font-bold mb-4 text-gray-800">📝 สร้างคำสั่งงานใหม่</h2>
            <form onsubmit="addAssignment(event)">
                <label class="block text-sm font-medium text-gray-700 mb-1">หัวข้องาน</label>
                <input type="text" id="assignTitle" required class="w-full px-4 py-2 border rounded-xl mb-3 focus:ring-2 focus:ring-indigo-500 outline-none" placeholder="เช่น รายงานเศรษฐศาสตร์เบื้องต้น">
                <label class="block text-sm font-medium text-gray-700 mb-1">รายละเอียด / คำชี้แจง</label>
                <textarea id="assignDesc" rows="3" required class="w-full px-4 py-2 border rounded-xl mb-3 focus:ring-2 focus:ring-indigo-500 outline-none" placeholder="ระบุรายละเอียด..."></textarea>
                <div class="grid grid-cols-2 gap-3 mb-6">
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">กำหนดส่ง (วันที่)</label>
                        <input type="date" id="assignDate" required class="w-full px-3 py-2 border rounded-xl text-sm focus:ring-2 focus:ring-indigo-500 outline-none">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">เวลา</label>
                        <input type="time" id="assignTime" value="23:59" required class="w-full px-3 py-2 border rounded-xl text-sm focus:ring-2 focus:ring-indigo-500 outline-none">
                    </div>
                </div>
                <div class="flex justify-end space-x-3">
                    <button type="button" onclick="closeAssignmentModal()" class="px-4 py-2 text-gray-600 hover:bg-gray-100 rounded-xl text-sm">ยกเลิก</button>
                    <button type="submit" class="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl text-sm font-medium shadow-md">มอบหมายงาน</button>
                </div>
            </form>
        </div>
    </div>

    <!-- JavaScript Logic -->
    <script>
        let currentRole = 'teacher'; 
        let teacherPassword = localStorage.getItem('teacherPass') || 'teacher123';
        let currentStudentName = ''; 
        let currentStudentId = '';
        
        let classStudents = {};
        let submissions = {};
        let streamPosts = {};

        let globalNotifications = []; 
        let activeRoomCode = ''; 
        let activeCheckAssignId = '';
        let gradingStudentName = '';

        function switchRole(role, studentName = '', studentId = '') {
            currentRole = role;
            const createClassBtn = document.getElementById('createClassBtn');
            const joinClassBtn = document.getElementById('joinClassBtn');
            const emptyCreateBtn = document.getElementById('emptyCreateBtn');
            const emptyJoinBtn = document.getElementById('emptyJoinBtn');
            const teacherControlBox = document.getElementById('teacherControlBox');
            const mainTitle = document.getElementById('mainTitle');
            const mainSubtitle = document.getElementById('mainSubtitle');
            const roleAvatar = document.getElementById('roleAvatar');
            const simulateStudentBtn = document.getElementById('simulateStudentBtn');
            const openTeacherLoginBtn = document.getElementById('openTeacherLoginBtn');
            const setupTeacherPassBtn = document.getElementById('setupTeacherPassBtn');
            const currentLoggedUser = document.getElementById('currentLoggedUser');
            const userNameDisplay = document.getElementById('userNameDisplay');
            const userIdDisplay = document.getElementById('userIdDisplay');

            if (role === 'teacher') {
                currentStudentName = '';
                currentStudentId = '';
                mainTitle.innerText = "ห้องเรียนของฉัน (มุมมองคุณครูผู้ควบคุม)";
                mainSubtitle.innerText = "จัดการห้องเรียน สร้างงาน ประกาศข่าวสาร และตรวจสอบข้อมูลการลงทะเบียนของนักเรียน";
                
                createClassBtn.classList.remove('hidden');
                joinClassBtn.classList.add('hidden');
                emptyCreateBtn.classList.remove('hidden');
                emptyJoinBtn.classList.add('hidden');

                roleAvatar.innerText = "คร";
                roleAvatar.className = "w-10 h-10 rounded-full bg-indigo-200 flex items-center justify-center font-bold text-indigo-700 shadow-inner";
                
                teacherControlBox?.classList.remove('hidden');
                simulateStudentBtn?.classList.remove('hidden');
                openTeacherLoginBtn?.classList.add('hidden');
                setupTeacherPassBtn?.classList.remove('hidden');
                currentLoggedUser?.classList.add('hidden');

                document.querySelectorAll('.teacher-only-feature').forEach(el => el.classList.remove('hidden'));
                document.querySelectorAll('.student-only-feature').forEach(el => el.classList.add('hidden'));
                document.querySelectorAll('.student-submit-box').forEach(el => el.classList.add('hidden'));
            } else {
                currentStudentName = studentName;
                currentStudentId = studentId;
                userNameDisplay.innerText = studentName;
                userIdDisplay.innerText = studentId;

                mainTitle.innerText = "ห้องเรียนของฉัน (มุมมองนักเรียน)";
                mainSubtitle.innerText = "เข้าเรียน ส่งงาน และติดตามประกาศจากคุณครู";
                
                createClassBtn.classList.add('hidden');
                joinClassBtn.classList.remove('hidden');
                emptyCreateBtn.classList.add('hidden');
                emptyJoinBtn.classList.remove('hidden');

                roleAvatar.innerText = "นร";
                roleAvatar.className = "w-10 h-10 rounded-full bg-pink-200 flex items-center justify-center font-bold text-pink-700 shadow-inner";

                teacherControlBox?.classList.add('hidden');
                simulateStudentBtn?.classList.add('hidden');
                openTeacherLoginBtn?.classList.remove('hidden');
                setupTeacherPassBtn?.classList.add('hidden');
                currentLoggedUser?.classList.remove('hidden');

                document.querySelectorAll('.teacher-only-feature').forEach(el => el.classList.add('hidden'));
                document.querySelectorAll('.student-only-feature').forEach(el => el.classList.remove('hidden'));
                document.querySelectorAll('.student-submit-box').forEach(el => el.classList.remove('hidden'));
            }

            filterClassroomsVisibility();
            if (activeRoomCode) {
                renderStreamPosts();
                updateStudentSubmitUI();
            }
        }

        function openCreateModal() {
            if (currentRole !== 'teacher') {
                alert('❌ เฉพาะคุณครูเท่านั้นที่สามารถสร้างห้องเรียนได้');
                return;
            }
            document.getElementById('createModal').classList.remove('hidden');
        }

        function closeCreateModal() {
            document.getElementById('createModal').classList.add('hidden');
        }

        function checkEmptyState() {
            const emptyState = document.getElementById('empty-state');
            const cards = document.querySelectorAll('.class-item');
            let visibleCount = 0;

            cards.forEach(card => {
                if (!card.classList.contains('hidden')) {
                    visibleCount++;
                }
            });

            if (visibleCount === 0) {
                emptyState.classList.remove('hidden');
                const emptyDesc = document.getElementById('emptyStateDesc');
                if (currentRole === 'teacher') {
                    emptyDesc.innerText = "เริ่มต้นใช้งานโดยการกดปุ่มสร้างห้องเรียนใหม่ด้านบน";
                } else {
                    emptyDesc.innerText = "คุณยังไม่ได้ลงทะเบียนในห้องเรียนใดๆ กดปุ่มด้านบนเพื่อเข้าร่วมห้องเรียน";
                }
            } else {
                emptyState.classList.add('hidden');
            }
        }

        function filterClassroomsVisibility() {
            const cards = document.querySelectorAll('.class-item');
            cards.forEach(card => {
                const code = card.getAttribute('data-code');
                if (currentRole === 'teacher') {
                    card.classList.remove('hidden');
                } else {
                    const isRegistered = classStudents[code]?.some(s => s.name === currentStudentName && s.id === currentStudentId);
                    if (isRegistered) {
                        card.classList.remove('hidden');
                    } else {
                        card.classList.add('hidden');
                    }
                }
            });

            checkEmptyState();
        }

        function openJoinModal() {
            document.getElementById('joinModal').classList.remove('hidden');
        }

        function closeJoinModal() {
            document.getElementById('joinModal').classList.add('hidden');
        }

        function registerAndJoinClassroom(e) {
            e.preventDefault();
            
            const studentId = document.getElementById('studentIdInput').value.trim();
            const studentName = document.getElementById('studentNameInput').value.trim();
            const code = document.getElementById('joinCodeInput').value.trim().toUpperCase();

            const targetCard = document.querySelector(`.class-item[data-code="${code}"]`);
            
            if (!targetCard) {
                alert('❌ ไม่พบรหัสห้องเรียนนี้ในระบบ กรุณาตรวจสอบรหัสใหม่อีกครั้ง');
                return;
            }

            if (!classStudents[code]) {
                classStudents[code] = [];
            }

            const exists = classStudents[code].some(s => s.id === studentId || s.name === studentName);
            if (!exists) {
                classStudents[code].push({ id: studentId, name: studentName });
                
                const countElem = targetCard.querySelector('.student-count');
                if (countElem) {
                    countElem.innerText = classStudents[code].length;
                }

                addNotification(`🎉 คุณได้ลงทะเบียนเข้าเรียนห้อง [${code}] เรียบร้อยแล้ว`);
            }

            switchRole('student', studentName, studentId);
            closeJoinModal();
            
            const title = targetCard.querySelector('h3').innerText;
            openClassroom(title, code);
        }

        function openClassroom(title, code) {
            activeRoomCode = code;
            document.getElementById('roomTitle').innerText = title;
            document.getElementById('roomCode').innerText = `รหัสห้องเรียน: ${code}`;
            
            const studentCount = classStudents[code] ? classStudents[code].length : 0;
            document.getElementById('modalStudentCount').innerText = studentCount;

            switchRoomTab('stream');
            renderStreamPosts();
            updateStudentSubmitUI();
            
            document.getElementById('roomModal').classList.remove('hidden');
        }

        function closeClassroom() {
            activeRoomCode = '';
            document.getElementById('roomModal').classList.add('hidden');
        }

        function switchRoomTab(tab) {
            const streamTab = document.getElementById('roomTabStream');
            const assignTab = document.getElementById('roomTabAssignments');
            const streamBtn = document.getElementById('tabStreamBtn');
            const assignBtn = document.getElementById('tabAssignmentsBtn');

            if (tab === 'stream') {
                streamTab.classList.remove('hidden');
                assignTab.classList.add('hidden');
                streamBtn.className = "relative px-4 py-2 rounded-xl font-bold text-sm bg-indigo-600 text-white shadow transition";
                assignBtn.className = "relative px-4 py-2 rounded-xl font-bold text-sm bg-gray-100 text-gray-600 hover:bg-gray-200 transition";
            } else {
                streamTab.classList.add('hidden');
                assignTab.classList.remove('hidden');
                assignBtn.className = "relative px-4 py-2 rounded-xl font-bold text-sm bg-indigo-600 text-white shadow transition";
                streamBtn.className = "relative px-4 py-2 rounded-xl font-bold text-sm bg-gray-100 text-gray-600 hover:bg-gray-200 transition";
            }
        }

        function postToStream() {
            const input = document.getElementById('streamPostInput');
            const text = input.value.trim();
            if (!text) return;

            if (!streamPosts[activeRoomCode]) streamPosts[activeRoomCode] = [];

            const author = currentRole === 'teacher' ? '👩‍🏫 คุณครู' : `👤 ${currentStudentName}`;
            const now = new Date();
            const timeStr = `${now.getDate()} ก.ย. ${now.getFullYear() + 543} เวลา ${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')} น.`;

            streamPosts[activeRoomCode].unshift({
                author: author,
                role: currentRole,
                time: timeStr,
                text: text
            });

            input.value = '';
            renderStreamPosts();
        }

        function renderStreamPosts() {
            const container = document.getElementById('streamPostsContainer');
            container.innerHTML = '';

            const posts = streamPosts[activeRoomCode] || [];
            if (posts.length === 0) {
                container.innerHTML = '<p class="text-xs text-gray-400 text-center py-4">ยังไม่มีประกาศหรือข้อความในกระดานนี้</p>';
                return;
            }

            posts.forEach(post => {
                const card = document.createElement('div');
                card.className = "bg-white p-4 rounded-2xl border border-gray-100 shadow-sm space-y-1";
                card.innerHTML = `
                    <div class="flex justify-between items-center text-xs">
                        <span class="font-bold ${post.role === 'teacher' ? 'text-indigo-600' : 'text-pink-600'}">${post.author}</span>
                        <span class="text-gray-400">${post.time}</span>
                    </div>
                    <p class="text-sm text-gray-700 whitespace-pre-wrap mt-1">${post.text}</p>
                `;
                container.appendChild(card);
            });
        }

        function submitAssignment(assignId) {
            if (currentRole !== 'student') {
                alert('เฉพาะนักเรียนเท่านั้นที่สามารถส่งงานได้');
                return;
            }

            const fileInput = document.getElementById(`file-${assignId}`);
            const file = fileInput ? fileInput.files[0] : null;
            const fileName = file ? file.name : 'งานที่ส่งออนไลน์.pdf';

            if (!submissions[activeRoomCode]) submissions[activeRoomCode] = {};
            if (!submissions[activeRoomCode][assignId]) submissions[activeRoomCode][assignId] = {};

            const assignCard = document.querySelector(`.assignment-card[data-id="${assignId}"]`);
            const dueDateStr = assignCard?.getAttribute('data-duedate');
            const now = new Date();
            const isLate = dueDateStr ? now > new Date(dueDateStr) : false;

            const timeStr = `${now.getDate()} ก.ย. ${now.getFullYear() + 543} เวลา ${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')} น.`;

            const reader = new FileReader();
            const processSubmission = (fileData = null) => {
                submissions[activeRoomCode][assignId][currentStudentName] = {
                    status: 'submitted',
                    time: timeStr,
                    timestamp: now.getTime(),
                    fileName: fileName,
                    fileData: fileData,
                    isLate: isLate,
                    grade: null,
                    comment: null
                };

                updateStudentSubmitUI();
                addNotification(`📤 คุณได้ส่งงานในวิชา [${activeRoomCode}] เรียบร้อยแล้ว`);
                alert('✅ ส่งงานเรียบร้อยแล้ว!');
            };

            if (file) {
                reader.onload = function(e) {
                    processSubmission(e.target.result);
                };
                reader.readAsDataURL(file);
            } else {
                processSubmission();
            }
        }

        function updateStudentSubmitUI() {
            if (currentRole !== 'student' || !activeRoomCode) return;

            const roomSubmissions = submissions[activeRoomCode] || {};
            
            document.querySelectorAll('.assignment-card').forEach(card => {
                const assignId = card.getAttribute('data-id');
                const subData = roomSubmissions[assignId] ? roomSubmissions[assignId][currentStudentName] : null;

                const statusElem = document.getElementById(`status-${assignId}`);
                const feedbackElem = document.getElementById(`feedback-${assignId}`);
                const actionBtnBox = document.getElementById(`actionBtnBox-${assignId}`);

                if (subData) {
                    let statusHtml = `<span class="text-green-600 font-bold">✓ ส่งแล้ว (${subData.time})</span>`;
                    if (subData.isLate) {
                        statusHtml += ` <span class="text-red-500 font-bold text-[10px]">(ส่งช้า)</span>`;
                    }
                    if (statusElem) statusElem.innerHTML = `สถานะ: ${statusHtml}`;

                    if (subData.grade || subData.comment) {
                        if (feedbackElem) {
                            feedbackElem.classList.remove('hidden');
                            feedbackElem.innerHTML = `
                                <div class="font-bold text-indigo-700">💯 คะแนน: ${subData.grade || 'รอตรวจ'}</div>
                                ${subData.comment ? `<div class="mt-1 text-gray-600">💬 ครู: ${subData.comment}</div>` : ''}
                            `;
                        }
                    } else if (feedbackElem) {
                        feedbackElem.classList.add('hidden');
                    }

                    if (actionBtnBox) {
                        actionBtnBox.innerHTML = `
                            <button onclick="submitAssignment('${assignId}')" class="w-full bg-gray-200 hover:bg-gray-300 text-gray-700 py-1.5 rounded-lg text-xs font-medium transition">
                                ส่งใหม่อีกครั้ง (Resubmit)
                            </button>
                        `;
                    }
                } else {
                    if (statusElem) statusElem.innerHTML = `<span class="text-gray-500">สถานะ: ยังไม่ได้ส่ง</span>`;
                    if (feedbackElem) feedbackElem.classList.add('hidden');
                    if (actionBtnBox) {
                        actionBtnBox.innerHTML = `
                            <button onclick="submitAssignment('${assignId}')" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white py-1.5 rounded-lg text-xs font-medium shadow">
                                ส่งงาน (Turn In)
                            </button>
                        `;
                    }
                }
            });
        }

        function openCheckSubmissionsModal(assignId, title) {
            activeCheckAssignId = assignId;
            document.getElementById('checkAssignTitle').innerText = `หัวข้องาน: ${title}`;

            const container = document.getElementById('submissionsListContainer');
            container.innerHTML = '';

            const registered = classStudents[activeRoomCode] || [];
            const roomSubs = (submissions[activeRoomCode] && submissions[activeRoomCode][assignId]) ? submissions[activeRoomCode][assignId] : {};

            if (registered.length === 0) {
                container.innerHTML = '<p class="text-xs text-gray-400 text-center py-4">ยังไม่มีนักเรียนลงทะเบียนในห้องนี้</p>';
            } else {
                registered.forEach(student => {
                    const sub = roomSubs[student.name];
                    const item = document.createElement('div');
                    item.className = "flex justify-between items-center bg-white p-3 rounded-xl border border-gray-100 shadow-sm";

                    let statusBadge = `<span class="text-xs bg-gray-100 text-gray-500 px-2 py-1 rounded-lg">ยังไม่ส่ง</span>`;
                    let actionBtn = ``;

                    if (sub) {
                        statusBadge = `<span class="text-xs bg-green-100 text-green-700 px-2 py-1 rounded-lg font-bold">ส่งแล้ว ${sub.isLate ? '(ช้า)' : ''}</span>`;
                        actionBtn = `
                            <button onclick="openGradeStudentModal('${student.name}')" class="bg-indigo-50 hover:bg-indigo-100 text-indigo-600 px-2.5 py-1 rounded-lg text-xs font-bold transition">
                                ${sub.grade ? '✏️ แก้ไขคะแนน' : '✍️ ให้คะแนน'}
                            </button>
                        `;
                    }

                    item.innerHTML = `
                        <div>
                            <div class="text-sm font-bold text-gray-800">${student.name}</div>
                            <div class="text-xs text-gray-400">รหัส: ${student.id}</div>
                        </div>
                        <div class="flex items-center space-x-2">
                            ${statusBadge}
                            ${actionBtn}
                        </div>
                    `;
                    container.appendChild(item);
                });
            }

            document.getElementById('checkSubmissionsModal').classList.remove('hidden');
        }

        function closeCheckSubmissionsModal() {
            document.getElementById('checkSubmissionsModal').classList.add('hidden');
        }

        function openGradeStudentModal(studentName) {
            gradingStudentName = studentName;
            const sub = submissions[activeRoomCode][activeCheckAssignId][studentName];

            document.getElementById('gradeModalStudentName').innerText = `นักเรียน: ${studentName}`;
            document.getElementById('gradeModalFileName').innerText = sub.fileName || 'ไฟล์แนบ.pdf';
            document.getElementById('gradeModalTime').innerText = sub.time;
            document.getElementById('inputGradeScore').value = sub.grade || '';
            document.getElementById('inputPrivateComment').value = sub.comment || '';

            document.getElementById('gradeStudentModal').classList.remove('hidden');
        }

        function closeGradeStudentModal() {
            document.getElementById('gradeStudentModal').classList.add('hidden');
        }

        function saveGradeAndFeedback() {
            const score = document.getElementById('inputGradeScore').value.trim();
            const comment = document.getElementById('inputPrivateComment').value.trim();

            if (submissions[activeRoomCode] && submissions[activeRoomCode][activeCheckAssignId] && submissions[activeRoomCode][activeCheckAssignId][gradingStudentName]) {
                const sub = submissions[activeRoomCode][activeCheckAssignId][gradingStudentName];
                sub.grade = score;
                sub.comment = comment;

                addNotification(`📝 ครูได้ตรวจงานและให้คะแนนแก่ ${gradingStudentName} แล้ว`);
                alert('💾 บันทึกคะแนนเรียบร้อยแล้ว');
            }

            closeGradeStudentModal();
            openCheckSubmissionsModal(activeCheckAssignId, document.getElementById('checkAssignTitle').innerText.replace('หัวข้องาน: ', ''));
        }

        function openPreviewWork() {
            const sub = submissions[activeRoomCode]?.[activeCheckAssignId]?.[gradingStudentName];
            const container = document.getElementById('previewContentContainer');
            document.getElementById('previewFileNameTitle').innerText = sub?.fileName || 'ไฟล์งาน';

            if (sub?.fileData) {
                if (sub.fileData.startsWith('data:image')) {
                    container.innerHTML = `<img src="${sub.fileData}" class="max-h-[60vh] object-contain rounded-lg shadow" />`;
                } else {
                    container.innerHTML = `<iframe src="${sub.fileData}" class="w-full h-[60vh] rounded-lg"></iframe>`;
                }
            } else {
                container.innerHTML = `
                    <div class="text-center p-8 text-gray-500">
                        <div class="text-4xl mb-2">📄</div>
                        <p class="font-bold text-sm">ตัวอย่างไฟล์งานแบบจำลอง (${sub?.fileName || 'document.pdf'})</p>
                        <p class="text-xs text-gray-400 mt-1">แสดงผลตัวอย่างเอกสารการเรียนและรายงานของนักเรียน</p>
                    </div>
                `;
            }

            document.getElementById('filePreviewModal').classList.remove('hidden');
        }

        function closeFilePreviewModal() {
            document.getElementById('filePreviewModal').classList.add('hidden');
        }

        function openStudentListModal() {
            const container = document.getElementById('studentListContainer');
            container.innerHTML = '';

            const list = classStudents[activeRoomCode] || [];
            if (list.length === 0) {
                container.innerHTML = '<p class="text-xs text-gray-400 text-center py-4">ยังไม่มีนักเรียนลงทะเบียน</p>';
            } else {
                list.forEach((student, index) => {
                    const item = document.createElement('div');
                    item.className = "flex justify-between items-center bg-white p-2.5 rounded-xl border border-gray-100 text-xs";
                    item.innerHTML = `
                        <span class="font-bold text-gray-700">${index + 1}. ${student.name}</span>
                        <span class="text-indigo-600 font-mono bg-indigo-50 px-2 py-0.5 rounded">ID: ${student.id}</span>
                    `;
                    container.appendChild(item);
                });
            }

            document.getElementById('studentListModal').classList.remove('hidden');
        }

        function closeStudentListModal() {
            document.getElementById('studentListModal').classList.add('hidden');
        }

        function addClassroom(e) {
            e.preventDefault();
            if (currentRole !== 'teacher') return;

            const name = document.getElementById('className').value.trim();
            const teacher = document.getElementById('teacherName').value.trim();
            const code = 'CS' + Math.floor(100 + Math.random() * 900);

            classStudents[code] = [];
            
            const grid = document.getElementById('classroom-grid');
            const card = document.createElement('div');
            card.className = "glass rounded-2xl shadow-xl overflow-hidden flex flex-col justify-between class-item";
            card.setAttribute('data-code', code);
            card.setAttribute('data-joined', 'true');

            card.innerHTML = `
                <div class="bg-gradient-to-r from-purple-500 to-pink-600 p-6 text-white relative">
                    <h3 class="text-xl font-bold">${name}</h3>
                    <p class="text-purple-100 text-sm mt-1">${teacher}</p>
                    <span class="absolute top-4 right-4 bg-white/20 text-xs px-2.5 py-1 rounded-full">รหัส: ${code}</span>
                </div>
                <div class="p-6 flex-1 flex flex-col justify-between">
                    <div>
                        <div class="flex justify-between text-sm text-gray-600 mb-4">
                            <span>นักเรียนลงทะเบียนแล้ว: <strong class="student-count text-indigo-600">0</strong> คน</span>
                            <span class="text-indigo-600 font-semibold">สถานะ: เปิดอยู่</span>
                        </div>
                    </div>
                    <div class="space-y-2">
                        <button onclick="openClassroom('${name}', '${code}')" class="w-full bg-indigo-50 hover:bg-indigo-100 text-indigo-600 border border-indigo-200 py-2 rounded-xl font-medium text-sm transition">
                            เข้าสู่ห้องเรียน
                        </button>
                        <button onclick="deleteClassroom('${code}')" class="w-full teacher-only-feature bg-red-50 hover:bg-red-100 text-red-600 border border-red-200 py-1.5 rounded-xl font-medium text-xs transition">
                            🗑️ ลบห้องเรียนนี้
                        </button>
                        <button onclick="leaveClassroom('${code}')" class="w-full student-only-feature bg-amber-50 hover:bg-amber-100 text-amber-700 border border-amber-200 py-1.5 rounded-xl font-medium text-xs transition hidden">
                            🚪 ถอนตัวออกจากห้องเรียน
                        </button>
                    </div>
                </div>
            `;

            grid.appendChild(card);
            closeCreateModal();
            addNotification(`✨ คุณครูได้สร้างห้องเรียนใหม่ [${name}] (รหัส: ${code})`);
            checkEmptyState();
        }

        function deleteClassroom(code) {
            if (confirm(`คุณต้องการลบห้องเรียนรหัส ${code} ใช่หรือไม่?`)) {
                const card = document.querySelector(`.class-item[data-code="${code}"]`);
                if (card) card.remove();
                delete classStudents[code];
                delete submissions[code];
                delete streamPosts[code];
                addNotification(`🗑️ ลบห้องเรียนรหัส [${code}] เรียบร้อยแล้ว`);
                checkEmptyState();
            }
        }

        function leaveClassroom(code) {
            if (confirm(`คุณต้องการถอนตัวออกจากห้องเรียนรหัส ${code} ใช่หรือไม่?`)) {
                if (classStudents[code]) {
                    classStudents[code] = classStudents[code].filter(s => s.name !== currentStudentName || s.id !== currentStudentId);
                }
                filterClassroomsVisibility();
                addNotification(`🚪 คุณได้ถอนตัวออกจากห้องเรียน [${code}] แล้ว`);
            }
        }

        function openAssignmentModal() {
            document.getElementById('assignmentModal').classList.remove('hidden');
        }

        function closeAssignmentModal() {
            document.getElementById('assignmentModal').classList.add('hidden');
        }

        function addAssignment(e) {
            e.preventDefault();
            const title = document.getElementById('assignTitle').value.trim();
            const desc = document.getElementById('assignDesc').value.trim();
            const date = document.getElementById('assignDate').value;
            const time = document.getElementById('assignTime').value;

            const assignId = 'assign-' + Date.now();
            const assignList = document.getElementById('assignment-list');

            const card = document.createElement('div');
            card.className = "bg-white p-5 rounded-2xl border border-gray-200 shadow-sm flex flex-col md:flex-row justify-between gap-4 assignment-card";
            card.setAttribute('data-id', assignId);
            card.setAttribute('data-duedate', `${date}T${time}`);

            card.innerHTML = `
                <div class="flex-1">
                    <span class="bg-indigo-100 text-indigo-700 text-xs px-2.5 py-1 rounded-full font-medium">การบ้าน</span>
                    <h3 class="font-bold text-gray-900 text-lg mt-2 assign-title-text">${title}</h3>
                    <p class="text-sm text-gray-600 mt-1">${desc}</p>
                    <p class="text-xs text-red-500 font-semibold mt-3">🕒 กำหนดส่ง: ${date} เวลา ${time} น.</p>
                    
                    <button onclick="openCheckSubmissionsModal('${assignId}', '${title}')" class="mt-4 teacher-only-feature bg-amber-100 hover:bg-amber-200 text-amber-800 text-xs font-bold px-3 py-1.5 rounded-lg border border-amber-200 transition">
                        📊 ตรวจสถานะและให้คะแนน
                    </button>
                </div>

                <div class="w-full md:w-72 bg-gray-50 p-4 rounded-xl border border-gray-100 flex flex-col justify-between student-submit-box ${currentRole === 'teacher' ? 'hidden' : ''}">
                    <div>
                        <h4 class="font-bold text-xs text-gray-700 mb-2">📤 ส่งงานของคุณ</h4>
                        <div id="uploadBox-${assignId}">
                            <input type="file" id="file-${assignId}" accept="image/*,.pdf,.doc,.docx" class="text-xs text-gray-500 file:mr-2 file:py-1.5 file:px-3 file:rounded-lg file:border-0 file:text-xs file:font-semibold file:bg-indigo-50 file:text-indigo-700 mb-2 w-full">
                        </div>
                        <div id="status-${assignId}" class="text-xs font-semibold text-gray-500 mb-2">สถานะ: ยังไม่ได้ส่ง</div>
                        <div id="feedback-${assignId}" class="text-xs bg-indigo-50 p-2 rounded-lg text-indigo-900 hidden mb-2 border border-indigo-100"></div>
                    </div>
                    <div id="actionBtnBox-${assignId}">
                        <button onclick="submitAssignment('${assignId}')" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white py-1.5 rounded-lg text-xs font-medium shadow">
                            ส่งงาน (Turn In)
                        </button>
                    </div>
                </div>
            `;

            assignList.prepend(card);
            closeAssignmentModal();
            addNotification(`📢 มีคำสั่งงานใหม่ในวิชา [${activeRoomCode}]: ${title}`);
        }

        function switchToStudentPrompt() {
            document.getElementById('studentPromptModal').classList.remove('hidden');
        }

        function closeStudentPromptModal() {
            document.getElementById('studentPromptModal').classList.add('hidden');
        }

        function confirmStudentSimulation(e) {
            e.preventDefault();
            const id = document.getElementById('simulateIdInput').value.trim();
            const name = document.getElementById('simulateNameInput').value.trim();

            switchRole('student', name, id);
            closeStudentPromptModal();
        }

        function openTeacherLoginModal() {
            document.getElementById('teacherLoginModal').classList.remove('hidden');
        }

        function closeTeacherLoginModal() {
            document.getElementById('teacherLoginModal').classList.add('hidden');
        }

        function verifyTeacherPassword(e) {
            e.preventDefault();
            const pass = document.getElementById('teacherPassInput').value;
            if (pass === teacherPassword) {
                switchRole('teacher');
                closeTeacherLoginModal();
                document.getElementById('teacherPassInput').value = '';
            } else {
                alert('❌ รหัสผ่านไม่ถูกต้อง');
            }
        }

        function openSetupPasswordModal() {
            document.getElementById('setupPasswordModal').classList.remove('hidden');
        }

        function closeSetupPasswordModal() {
            document.getElementById('setupPasswordModal').classList.add('hidden');
        }

        function saveTeacherPassword(e) {
            e.preventDefault();
            const newPass = document.getElementById('newTeacherPass').value;
            teacherPassword = newPass;
            localStorage.setItem('teacherPass', newPass);
            alert('⚙️ บันทึกรหัสผ่านใหม่เรียบร้อยแล้ว');
            closeSetupPasswordModal();
        }

        function addNotification(msg) {
            const now = new Date();
            const timeStr = `${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')}`;
            globalNotifications.unshift({ text: msg, time: timeStr });

            const badge = document.getElementById('globalNotifBadge');
            badge.innerText = globalNotifications.length;
            badge.classList.remove('hidden');
        }

        function openNotificationModal() {
            const container = document.getElementById('notificationListContainer');
            container.innerHTML = '';

            if (globalNotifications.length === 0) {
                container.innerHTML = '<p class="text-xs text-gray-400 text-center py-6">ยังไม่มีการแจ้งเตือนใหม่ในขณะนี้</p>';
            } else {
                globalNotifications.forEach(n => {
                    const item = document.createElement('div');
                    item.className = "p-3 bg-gray-50 rounded-xl border border-gray-100 text-xs flex justify-between items-start";
                    item.innerHTML = `
                        <span class="text-gray-700 font-medium">${n.text}</span>
                        <span class="text-[10px] text-gray-400 ml-2 whitespace-nowrap">${n.time}</span>
                    `;
                    container.appendChild(item);
                });
            }

            document.getElementById('globalNotifBadge').classList.add('hidden');
            document.getElementById('notificationModal').classList.remove('hidden');
        }

        function closeNotificationModal() {
            document.getElementById('notificationModal').classList.add('hidden');
        }

        // เริ่มต้นการทำงานด้วยการอยู่ในมุมมองคุณครู
        switchRole('teacher');
    </script>
</body>
</html>
