<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hệ Thống Quản Lý Sinh Viên Chi Tiết</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        .error-cell { background-color: #fee2e2; border: 1px solid #ef4444 !important; }
        .input-focus { outline: none; border-bottom: 2px solid #4f46e5; }
    </tbody >
    </style>
</head>
<body class="bg-slate-50 p-4 md:p-8">

    <div class="max-w-7xl mx-auto bg-white shadow-2xl rounded-2xl overflow-hidden border border-slate-200">
        <div class="p-6 bg-indigo-600 text-white flex flex-wrap justify-between items-center gap-4">
            <div>
                <h1 class="text-2xl font-black">DANH SÁCH SINH VIÊN CHI TIẾT</h1>
                <p class="text-indigo-100 text-sm italic">Hỗ trợ: Họ tên, MSV, Ngày sinh, Lớp, Giới tính, Điểm</p>
            </div>
            <div class="flex gap-2">
                <input type="file" id="uploadFile" class="hidden" accept=".xlsx, .xls">
                <button onclick="document.getElementById('uploadFile').click()" class="bg-white text-indigo-600 px-4 py-2 rounded-lg font-bold hover:bg-indigo-50 transition">
                    📁 Tải Tệp Excel
                </button>
                <button onclick="addNewRow()" class="bg-indigo-500 text-white px-4 py-2 rounded-lg font-bold border border-indigo-400 hover:bg-indigo-400 transition">
                    + Thêm Dòng
                </button>
            </div>
        </div>

        <div class="overflow-x-auto">
            <table class="w-full border-collapse">
                <thead>
                    <tr class="bg-slate-100 text-slate-600 text-sm uppercase font-bold border-b">
                        <th class="p-4 text-center w-16">STT</th>
                        <th class="p-4">Họ và Tên</th>
                        <th class="p-4 w-32">MSV</th>
                        <th class="p-4 w-40">Ngày Sinh</th>
                        <th class="p-4 w-32 text-center">Giới Tính</th>
                        <th class="p-4 w-32">Lớp</th>
                        <th class="p-4 w-24 text-center">Điểm</th>
                        <th class="p-4 text-center w-24">Xử lý</th>
                    </tr>
                </thead>
                <tbody id="studentBody" class="divide-y divide-slate-200">
                    </tbody>
            </table>
        </div>

        <div id="statusLabel" class="p-4 bg-slate-50 text-slate-500 text-xs flex justify-between">
            <span>* Nhấn trực tiếp vào ô để sửa. Hệ thống tự lưu khi bạn nhập.</span>
            <span id="errorCount" class="text-red-500 font-bold"></span>
        </div>
    </div>

    <script>
        let students = JSON.parse(localStorage.getItem('fullStudentData')) || [
            { id: 1, name: "Nguyễn Văn An", msv: "SV001", dob: "2003-05-15", gender: "Nam", class: "IT01", score: 8.5 }
        ];

        // Đọc tệp Excel
        document.getElementById('uploadFile').addEventListener('change', function(e) {
            const file = e.target.files[0];
            const reader = new FileReader();
            reader.onload = (evt) => {
                const wb = XLSX.read(new Uint8Array(evt.target.result), {type: 'array'});
                const data = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]], {header: 1});
                
                // Chuyển đổi dữ liệu từ tệp (Bắt đầu từ dòng 2)
                const imported = data.slice(1).map((row, index) => ({
                    id: Date.now() + index,
                    name: row[0] || "",
                    msv: row[1] || "",
                    dob: row[2] || "",
                    gender: row[3] || "Nam",
                    class: row[4] || "",
                    score: parseFloat(row[5]) || 0
                }));
                
                students = [...students, ...imported];
                render();
            };
            reader.readAsArrayBuffer(file);
        });

        function addNewRow() {
            students.push({ id: Date.now(), name: "", msv: "", dob: "", gender: "Nam", class: "", score: 0 });
            render();
        }

        function updateData(id, field, value) {
            const index = students.findIndex(s => s.id === id);
            students[index][field] = field === 'score' ? parseFloat(value) || 0 : value;
            localStorage.setItem('fullStudentData', JSON.stringify(students));
            validateRow(id); // Kiểm tra lỗi ngay khi sửa
        }

        function deleteRow(id) {
            if(confirm("Xóa sinh viên này?")) {
                students = students.filter(s => s.id !== id);
                render();
            }
        }

        function validateRow(id) {
            // Logic kiểm tra lỗi đơn giản
            const s = students.find(item => item.id === id);
            const errors = [];
            if (!s.name) errors.push("Thiếu tên");
            if (s.score > 10 || s.score < 0) errors.push("Điểm sai");
            return errors;
        }

        function render() {
            const body = document.getElementById('studentBody');
            body.innerHTML = '';

            students.forEach((s, index) => {
                const isErrorScore = (s.score > 10 || s.score < 0);
                const isErrorName = s.name === "";

                const row = `
                    <tr class="hover:bg-indigo-50/50 transition">
                        <td class="p-4 text-center font-mono text-slate-400">${index + 1}</td>
                        <td class="p-2">
                            <input onchange="updateData(${s.id}, 'name', this.value)" value="${s.name}" 
                            class="w-full p-2 bg-transparent rounded ${isErrorName ? 'error-cell' : ''}" placeholder="Nhập tên...">
                        </td>
                        <td class="p-2">
                            <input onchange="updateData(${s.id}, 'msv', this.value)" value="${s.msv}" 
                            class="w-full p-2 bg-transparent font-mono text-indigo-600" placeholder="Mã số...">
                        </td>
                        <td class="p-2 text-center">
                            <input type="date" onchange="updateData(${s.id}, 'dob', this.value)" value="${s.dob}" 
                            class="w-full p-2 bg-transparent">
                        </td>
                        <td class="p-2">
                            <select onchange="updateData(${s.id}, 'gender', this.value)" class="w-full p-2 bg-transparent">
                                <option value="Nam" ${s.gender==='Nam'?'selected':''}>Nam</option>
                                <option value="Nữ" ${s.gender==='Nữ'?'selected':''}>Nữ</option>
                            </select>
                        </td>
                        <td class="p-2 text-center">
                            <input onchange="updateData(${s.id}, 'class', this.value)" value="${s.class}" 
                            class="w-full p-2 bg-transparent uppercase" placeholder="Lớp...">
                        </td>
                        <td class="p-2 text-center font-bold">
                            <input type="number" onchange="updateData(${s.id}, 'score', this.value)" value="${s.score}" 
                            class="w-20 p-2 text-center bg-transparent rounded ${isErrorScore ? 'error-cell' : 'text-emerald-600'}">
                        </td>
                        <td class="p-2 text-center">
                            <button onclick="deleteRow(${s.id})" class="text-red-400 hover:text-red-600 px-2">Xóa</button>
                        </td>
                    </tr>
                `;
                body.innerHTML += row;
            });
            localStorage.setItem('fullStudentData', JSON.stringify(students));
        }

        render();
    </script>
</body>
</html>
