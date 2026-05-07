<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>منصة النبراس - النسخة الاحترافية</title>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-firestore-compat.js"></script>
    <style>
        :root { --bg: #000; --red: #8b0000; --white: #fff; --gray: #222; }
        body, html { margin: 0; padding: 0; width: 100%; height: 100%; font-family: 'Segoe UI', sans-serif; background: var(--bg); color: var(--white); text-align: right; overflow-x: hidden; }
        
        .screen { display: none; padding: 20px; min-height: 100vh; width: 100%; box-sizing: border-box; }
        .active { display: block !important; }
        .flex-center { display: flex; flex-direction: column; justify-content: center; align-items: center; }

        .card { background: #111; padding: 20px; border-radius: 10px; border-right: 5px solid var(--red); margin-bottom: 20px; box-shadow: 0 4px 15px rgba(0,0,0,0.5); }
        .btn { padding: 12px 20px; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; width: 100%; margin: 5px 0; transition: 0.3s; }
        .btn-main { background: var(--red); color: white; }
        .btn-sub { background: white; color: black; }
        .btn-danger { background: #ff4444; color: white; }
        .btn-success { background: #28a745; color: white; }

        .row { display: flex; gap: 10px; width: 100%; }
        .row input { flex: 1; }
        
        input, select { width: 100%; padding: 12px; margin: 10px 0; background: #222; color: white; border: 1px solid #444; border-radius: 5px; box-sizing: border-box; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; font-size: 14px; }
        th, td { border: 1px solid #333; padding: 10px; text-align: center; }
        th { background: var(--red); }
        
        .option-btn { display: block; width: 100%; text-align: right; padding: 12px; margin: 5px 0; background: #222; border: 1px solid #444; color: white; cursor: pointer; border-radius: 5px; }
        .selected { background: var(--red) !important; border-color: white; }
        .footer-dev { font-size: 13px; margin-top: 15px; color: #aaa; text-align: center; }
    </style>
</head>
<body>

<!-- الرئيسية -->
<div id="home-screen" class="screen active flex-center" style="background: linear-gradient(135deg, #000 50%, #8b0000 50%);">
    <div style="background:rgba(0,0,0,0.9); padding:30px; border-radius:20px; border:2px solid #8b0000; width:320px; text-align:center;">
        <h2>منصة النبراس</h2>
        <button class="btn btn-sub" onclick="showScreen('student-login')">تسجيل دخول الطالب</button>
        <button class="btn btn-main" onclick="showScreen('student-register')">إنشاء حساب جديد</button>
        <button class="btn btn-sub" style="margin-top:20px; opacity:0.7" onclick="adminAuth()">دخول المستر</button>
        <p class="footer-dev">المطور: عمار ياسر - 01281872620</p>
    </div>
</div>

<!-- إنشاء حساب -->
<div id="student-register" class="screen flex-center">
    <div class="card" style="width: 350px;">
        <h3>بطل جديد في النبراس</h3>
        <div class="row">
            <input type="text" id="regName" placeholder="اسم الطالب">
            <input type="text" id="regFather" placeholder="اسم الأب">
        </div>
        <input type="tel" id="regPhone" placeholder="رقم موبايل الطالب">
        <input type="tel" id="regParentPhone" placeholder="رقم موبايل ولي الأمر">
        <button class="btn btn-main" onclick="handleRegister()">إنشاء الحساب</button>
        <button class="btn btn-sub" onclick="showScreen('home-screen')">رجوع</button>
    </div>
</div>

<!-- دخول الطالب -->
<div id="student-login" class="screen flex-center">
    <div class="card" style="width: 320px;">
        <h3>تسجيل دخول</h3>
        <input type="tel" id="loginPhone" placeholder="رقم الموبايل المسجل">
        <button class="btn btn-main" onclick="handleStudentLogin()">دخول</button>
        <button class="btn btn-sub" onclick="showScreen('home-screen')">رجوع</button>
    </div>
</div>

<!-- لوحة الطالب -->
<div id="student-dash" class="screen">
    <div class="card">
        <h2 id="stName">مرحباً بك</h2>
        <div id="links-area" style="margin-bottom: 15px;"></div>
        <button class="btn btn-main" id="startExamBtn" onclick="loadExam()">بدء الامتحان المتاح</button>
        <button class="btn btn-sub" onclick="location.reload()">خروج</button>
    </div>
    <div id="exam-area" style="display:none" class="card">
        <h3 id="exTitle">الأسئلة</h3>
        <div id="questions-box"></div>
        <button class="btn btn-main" onclick="submitExam()">إرسال الإجابات</button>
    </div>
</div>

<!-- لوحة المستر -->
<div id="admin-dash" class="screen">
    <h2>لوحة تحكم مستر أشرف فكري</h2>
    
    <!-- قسم إدارة الطلاب -->
    <div class="card">
        <h3>إدارة الطلاب (تفعيل/حظر)</h3>
        <div style="overflow-x: auto;">
            <table>
                <thead><tr><th>الطالب</th><th>الحالة</th><th>إجراء</th></tr></thead>
                <tbody id="usersTable"></tbody>
            </table>
        </div>
    </div>

    <!-- قسم الروابط -->
    <div class="card">
        <h3>إرسال لينكات للطلاب</h3>
        <input type="text" id="linkTitle" placeholder="عنوان الرابط (مثلاً: محاضرة النحو)">
        <input type="text" id="linkUrl" placeholder="ضع الرابط هنا">
        <button class="btn btn-success" onclick="addLink()">نشر الرابط للطلاب</button>
    </div>

    <!-- قسم الأسئلة -->
    <div class="card">
        <h3>إضافة سؤال</h3>
        <input type="text" id="qText" placeholder="نص السؤال">
        <input type="text" id="opt0" placeholder="الإجابة الصحيحة">
        <input type="text" id="opt1" placeholder="اختيار خطأ 1">
        <input type="text" id="opt2" placeholder="اختيار خطأ 2">
        <button class="btn btn-main" onclick="addQuestion()">حفظ السؤال</button>
        <hr>
        <h4>الأسئلة الحالية</h4>
        <div id="admin-questions-list"></div>
    </div>

    <button class="btn btn-sub" onclick="location.reload()">خروج نهائي</button>
</div>

<script>
    // --- إعدادات Firebase ---
    const firebaseConfig = {
      apiKey: "AIzaSyDDUR6IdJ-eQW72CN0pB0B6sqxBxZkefKg",
      authDomain: "al-nibras-in-the-arabic.firebaseapp.com",
      projectId: "al-nibras-in-the-arabic",
      storageBucket: "al-nibras-in-the-arabic.firebasestorage.app",
      messagingSenderId: "45491370927",
      appId: "1:45491370927:web:55dfc130d71b2257e1cd5a"
    };
    
    firebase.initializeApp(firebaseConfig);
    const db = firebase.firestore();

    let currentStudent = null;
    let questions = [];
    let answers = {};

    function showScreen(id) {
        document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
        const target = document.getElementById(id);
        if (target) {
            target.classList.add('active');
            window.scrollTo(0,0);
        }
    }

    // --- نظام الطلاب الجدد ---
    async function handleRegister() {
        const name = document.getElementById('regName').value.trim();
        const father = document.getElementById('regFather').value.trim();
        const phone = document.getElementById('regPhone').value.trim();
        const pPhone = document.getElementById('regParentPhone').value.trim();

        if(!name || !father || !phone || !pPhone) return alert("من فضلك أكمل جميع البيانات");

        try {
            await db.collection("student_requests").add({
                name: name + " " + father,
                studentPhone: phone,
                parentPhone: pPhone,
                status: "pending", // بانتظار التفعيل
                regDate: new Date().toLocaleString('ar-EG')
            });
            alert("حسابك إتعمل.. إبعت للمستر يفعله يا بطل");
            showScreen('home-screen');
        } catch (e) { alert("خطأ في التسجيل"); }
    }

    async function handleStudentLogin() {
        const phone = document.getElementById('loginPhone').value.trim();
        if(!phone) return alert("ادخل رقمك");

        const res = await db.collection("student_requests").where("studentPhone", "==", phone).get();
        if(res.empty) return alert("الرقم غير مسجل");

        const user = res.docs[0].data();
        if(user.status === "pending") return alert("حسابك لسه مفعلوش المستر.. تواصل معه");
        if(user.status === "blocked") return alert("تم حظرك من المنصة.. راجع المستر");

        currentStudent = { id: res.docs[0].id, ...user };
        document.getElementById('stName').innerText = "البطل: " + currentStudent.name;
        showScreen('student-dash');
        loadLinks();
    }

    // --- إدارة الروابط ---
    async function addLink() {
        const title = document.getElementById('linkTitle').value;
        const url = document.getElementById('linkUrl').value;
        if(!title || !url) return alert("اكمل بيانات الرابط");
        await db.collection("links").add({ title, url, date: new Date() });
        alert("تم النشر");
    }

    async function loadLinks() {
        const res = await db.collection("links").orderBy("date", "desc").get();
        const area = document.getElementById('links-area');
        area.innerHTML = "<h4>روابط هامة:</h4>";
        res.forEach(doc => {
            const d = doc.data();
            area.innerHTML += `<a href="${d.url}" target="_blank" style="color:#ff4444; display:block; margin:5px 0;">🔗 ${d.title}</a>`;
        });
    }

    // --- إدارة المستر ---
    function adminAuth() {
        const pass = prompt("كلمة سر المستر:");
        if(pass === "أشرف فكري") { 
            showScreen('admin-dash');
            loadUsers();
            loadAdminQuestions();
        }
    }

    async function loadUsers() {
        const res = await db.collection("student_requests").get();
        const table = document.getElementById('usersTable');
        table.innerHTML = "";
        res.forEach(doc => {
            const u = doc.data();
            const id = doc.id;
            let btn = u.status === "active" ? 
                `<button class="btn btn-danger" onclick="updateStatus('${id}', 'blocked')">حظر</button>` :
                `<button class="btn btn-success" onclick="updateStatus('${id}', 'active')">تفعيل/إلغاء حظر</button>`;
            
            table.innerHTML += `<tr>
                <td>${u.name}<br><small>${u.studentPhone}</small></td>
                <td>${u.status}</td>
                <td>${btn}</td>
            </tr>`;
        });
    }

    async function updateStatus(id, newStatus) {
        await db.collection("student_requests").doc(id).update({ status: newStatus });
        loadUsers();
    }

    // --- إدارة الأسئلة (إضافة وحذف) ---
    async function addQuestion() {
        const text = document.getElementById('qText').value;
        const correct = document.getElementById('opt0').value;
        const o1 = document.getElementById('opt1').value;
        const o2 = document.getElementById('opt2').value;
        if(!text || !correct) return alert("اكمل السؤال");

        await db.collection("questions").add({ text, options: [correct, o1, o2].sort(()=>Math.random()-0.5), correct });
        alert("تم الحفظ");
        loadAdminQuestions();
    }

    async function loadAdminQuestions() {
        const res = await db.collection("questions").get();
        const box = document.getElementById('admin-questions-list');
        box.innerHTML = "";
        res.forEach(doc => {
            const q = doc.data();
            box.innerHTML += `<div style="border-bottom:1px solid #333; padding:10px;">
                <span>${q.text}</span>
                <button class="btn btn-danger" style="width:auto; float:left; padding:5px 10px;" onclick="deleteQuestion('${doc.id}')">حذف</button>
                <div style="clear:both"></div>
            </div>`;
        });
    }

    async function deleteQuestion(id) {
        if(confirm("هل تريد حذف هذا السؤال نهائياً؟")) {
            await db.collection("questions").doc(id).delete();
            loadAdminQuestions();
        }
    }

    // --- منطق الامتحان ---
    async function loadExam() {
        const res = await db.collection("questions").get();
        questions = res.docs.map(doc => ({id: doc.id, ...doc.data()}));
        if(questions.length == 0) return alert("لا توجد أسئلة");
        
        document.getElementById('exam-area').style.display = 'block';
        document.getElementById('startExamBtn').style.display = 'none';
        const box = document.getElementById('questions-box');
        box.innerHTML = "";
        questions.forEach((q, i) => {
            let opts = "";
            q.options.forEach(opt => {
                opts += `<button class="option-btn q-${i}" onclick="selectOpt(${i}, '${opt.replace(/'/g, "\\'")}', this)">${opt}</button>`;
            });
            box.innerHTML += `<div class="card"><p><b>س ${i+1}:</b> ${q.text}</p>${opts}</div>`;
        });
    }

    function selectOpt(qi, val, btn) {
        answers[qi] = val;
        document.querySelectorAll('.q-'+qi).forEach(b => b.classList.remove('selected'));
        btn.classList.add('selected');
    }

    async function submitExam() {
        let score = 0;
        questions.forEach((q, i) => { if(answers[i] === q.correct) score++; });
        const total = Math.round((score / questions.length) * 100);
        await db.collection("results").add({
            name: currentStudent.name,
            grade: total + "%",
            date: new Date().toLocaleString('ar-EG')
        });
        alert("درجتك: " + total + "%");
        location.reload();
    }
</script>
</body>
</html>
