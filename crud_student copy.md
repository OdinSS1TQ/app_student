<!-- File: crud_student.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Student Management</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <!-- CSRF Token -->
    <script>
        window.csrf_token = "{{ frappe.session.csrf_token }}"; // Keep this if rendering via Frappe web route
        // window.csrf_token = "YOUR_CSRF_TOKEN"; // Or manually set if testing standalone (not recommended for production)
    </script>

    <!-- Bootstrap 5 CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">
    <!-- Bootstrap Icons -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">
    <!-- SweetAlert2 -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/sweetalert2@11/dist/sweetalert2.min.css">

    <style>
        body {
            background-color: #f8f9fa;
        }
        .container {
            max-width: 960px;
        }
        .card {
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            border: none;
        }
        .table thead {
            background-color: #e9ecef;
        }
        .btn-action {
            margin-right: 5px;
        }
        #alertPlaceholder {
            position: fixed;
            top: 10px;
            right: 10px;
            z-index: 1050;
            min-width: 300px;
        }
    </style>
</head>
<body>
    <div class="container mt-5 mb-5">
        <h1 class="text-center mb-4 text-primary">Student Management</h1>

        <!-- Alert Placeholder -->
        <div id="alertPlaceholder"></div>

        <!-- Student Form Card -->
        <div class="card mb-4">
            <div class="card-header bg-primary text-white">
                <h4 id="formTitle">Add New Student</h4>
            </div>
            <div class="card-body">
                <form id="studentForm">
                    <input type="hidden" id="student_name">
                    <div class="row g-3">
                        <div class="col-md-6">
                            <label for="student_id" class="form-label">Student ID <span class="text-danger">*</span></label>
                            <input type="text" class="form-control" id="student_id" required>
                        </div>
                        <div class="col-md-6">
                            <label for="full_name" class="form-label">Full Name <span class="text-danger">*</span></label>
                            <input type="text" class="form-control" id="full_name" required>
                        </div>
                        <div class="col-md-6">
                            <label for="gender" class="form-label">Gender <span class="text-danger">*</span></label>
                            <select class="form-select" id="gender" required>
                                <option selected disabled value="">Choose...</option>
                                <option>Male</option>
                                <option>Female</option>
                                <option>Other</option>
                            </select>
                        </div>
                        <div class="col-md-6">
                            <label for="date_of_birth" class="form-label">Date of Birth <span class="text-danger">*</span></label>
                            <input type="date" class="form-control" id="date_of_birth" required>
                        </div>
                        <div class="col-md-6">
                            <label for="enrollment_date" class="form-label">Enrollment Date <span class="text-danger">*</span></label>
                            <input type="date" class="form-control" id="enrollment_date" required>
                        </div>
                         <div class="col-md-6">
                            <label for="phone_number" class="form-label">Phone Number <span class="text-danger">*</span></label>
                            <input type="tel" class="form-control" id="phone_number" required pattern="[0-9\+\-\s()]*" title="Enter a valid phone number">
                        </div>
                         <div class="col-md-12">
                            <label for="email" class="form-label">Email <span class="text-danger">*</span></label>
                            <input type="email" class="form-control" id="email" required>
                        </div>
                    </div>
                    <div class="mt-4">
                        <button type="submit" id="saveButton" class="btn btn-success me-2">
                            <i class="bi bi-check-circle-fill"></i> Save
                        </button>
                        <button type="button" id="clearButton" class="btn btn-secondary">
                            <i class="bi bi-x-circle"></i> Clear
                        </button>
                    </div>
                </form>
            </div>
        </div>

        <!-- Student List Card -->
        <div class="card">
            <div class="card-header bg-light">
                <h4>Student List</h4>
            </div>
            <div class="card-body">
                <div class="table-responsive">
                    <table class="table table-hover align-middle">
                        <thead>
                            <tr>
                                <th>ID</th>
                                <th>Name</th>
                                <th>Gender</th>
                                <th>DOB</th>
                                <th>Enrolled</th>
                                <th>Phone</th>
                                <th>Email</th>
                                <th class="text-center">Actions</th>
                            </tr>
                        </thead>
                        <tbody id="studentsTable">
                            <tr>
                                <td colspan="8" class="text-center">Loading students...</td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>

    <!-- Axios -->
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <!-- Bootstrap 5 Bundle JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL" crossorigin="anonymous"></script>
    <!-- SweetAlert2 -->
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>

    <script>
        // --- Configuration ---
        const API_BASE_URL = '/api/resource/Student';
        axios.defaults.withCredentials = true; // Important for session handling
        axios.defaults.headers.common['X-Frappe-CSRF-Token'] = window.csrf_token;

        // --- DOM Elements ---
        const studentForm = document.getElementById('studentForm');
        const studentsTable = document.getElementById('studentsTable');
        const studentNameInput = document.getElementById('student_name');
        const studentIdInput = document.getElementById('student_id');
        const fullNameInput = document.getElementById('full_name');
        const genderInput = document.getElementById('gender');
        const dobInput = document.getElementById('date_of_birth');
        const enrollmentDateInput = document.getElementById('enrollment_date');
        const phoneInput = document.getElementById('phone_number');
        const emailInput = document.getElementById('email');
        const saveButton = document.getElementById('saveButton');
        const clearButton = document.getElementById('clearButton');
        const formTitle = document.getElementById('formTitle');
        const alertPlaceholder = document.getElementById('alertPlaceholder');

        // --- Utility Functions ---
        function showAlert(message, type = 'success') {
            // Types: success, danger, warning, info
            const wrapper = document.createElement('div');
            wrapper.innerHTML = `
                <div class="alert alert-${type} alert-dismissible fade show" role="alert">
                    ${message}
                    <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                </div>
            `;
            alertPlaceholder.append(wrapper);

            // Auto-dismiss after 5 seconds
            setTimeout(() => {
                wrapper.remove();
            }, 5000);
        }

        function clearForm() {
            studentForm.reset();
            studentNameInput.value = '';
            formTitle.textContent = 'Add New Student';
            saveButton.innerHTML = '<i class="bi bi-check-circle-fill"></i> Save';
            saveButton.disabled = false;
            studentIdInput.readOnly = false; // Make Student ID editable again when clearing
        }

        function setFormState(isEditing, docname = '') {
            if (isEditing) {
                formTitle.textContent = `Edit Student (${docname})`;
                saveButton.innerHTML = '<i class="bi bi-pencil-square"></i> Update';
                studentIdInput.readOnly = true; // Usually, the unique ID isn't editable
            } else {
                clearForm(); // Resets to 'Add New' state
            }
        }

        function disableSubmitButton(disabled = true) {
            saveButton.disabled = disabled;
            if (disabled) {
                const currentText = saveButton.textContent.trim();
                saveButton.innerHTML = `
                    <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
                    ${currentText}...
                `;
            } else {
                // Restore original text based on form state
                setFormState(!!studentNameInput.value, studentNameInput.value);
            }
        }

        // --- API Functions ---
        async function loadStudents() {
            studentsTable.innerHTML = '<tr><td colspan="8" class="text-center">Loading students... <span class="spinner-border spinner-border-sm"></span></td></tr>';
            try {
                const res = await axios.get(API_BASE_URL, {
                    params: {
                        fields: JSON.stringify([
                            "name", "student_id", "full_name", "gender", "date_of_birth",
                            "enrollment_date", "phone_number", "email"
                        ]),
                        limit_page_length: 100 // Fetch more if needed
                    }
                });

                if (res.data && Array.isArray(res.data.data)) {
                    const students = res.data.data;
                    if (students.length === 0) {
                        studentsTable.innerHTML = '<tr><td colspan="8" class="text-center text-muted">No students found. Add one using the form above!</td></tr>';
                        return;
                    }

                    let rows = students.map(s => {
                        const studentId = s.student_id || 'N/A';
                        const fullName = s.full_name || 'N/A';
                        const gender = s.gender || 'N/A';
                        const dob = s.date_of_birth || 'N/A';
                        const enrollDate = s.enrollment_date || 'N/A';
                        const phone = s.phone_number || 'N/A';
                        const email = s.email || 'N/A';
                        const studentName = s.name; // Frappe's internal document name

                        if (!studentName) {
                            console.warn("Student data missing 'name' property:", s);
                            return ''; // Skip row if essential ID is missing
                        }

                        return `
                            <tr>
                                <td>${studentId}</td>
                                <td>${fullName}</td>
                                <td>${gender}</td>
                                <td>${dob}</td>
                                <td>${enrollDate}</td>
                                <td>${phone}</td>
                                <td>${email}</td>
                                <td class="text-center">
                                    <button class="btn btn-sm btn-warning btn-action" onclick="editStudent('${studentName}')" title="Edit">
                                        <i class="bi bi-pencil-fill"></i>
                                    </button>
                                    <button class="btn btn-sm btn-danger btn-action" onclick="deleteStudent('${studentName}')" title="Delete">
                                        <i class="bi bi-trash-fill"></i>
                                    </button>
                                </td>
                            </tr>
                        `;
                    }).join('');
                    studentsTable.innerHTML = rows;
                } else {
                    throw new Error("Invalid data format received from API.");
                }
            } catch (err) {
                console.error('Error loading students:', err);
                const errorMsg = err.response?.data?.message || err.message || 'Failed to load students.';
                studentsTable.innerHTML = `<tr><td colspan="8" class="text-center text-danger">Error: ${errorMsg}</td></tr>`;
                showAlert(`Error loading students: ${errorMsg}`, 'danger');
            }
        }

        async function handleFormSubmit(e) {
            e.preventDefault();
            disableSubmitButton(true);

            const studentData = {
                student_id: studentIdInput.value.trim(),
                full_name: fullNameInput.value.trim(),
                gender: genderInput.value,
                date_of_birth: dobInput.value,
                enrollment_date: enrollmentDateInput.value,
                phone_number: phoneInput.value.trim(),
                email: emailInput.value.trim(),
                doctype: "Student" // Ensure doctype is specified for POST/PUT
            };

            const studentName = studentNameInput.value;
            let request;
            let successMessage;

            if (studentName) {
                // Update (PUT)
                request = axios.put(`${API_BASE_URL}/${studentName}`, studentData);
                successMessage = `Student '${studentData.full_name}' updated successfully!`;
            } else {
                // Create (POST)
                request = axios.post(API_BASE_URL, studentData);
                successMessage = `Student '${studentData.full_name}' added successfully!`;
            }

            try {
                await request;
                showAlert(successMessage, 'success');
                clearForm();
                loadStudents(); // Refresh the list
            } catch (err) {
                console.error('Error saving student:', err);
                 let errorMsg = 'Failed to save student.';
                 if (err.response) {
                    // Try to get Frappe specific error message
                    errorMsg = err.response.data?.message || err.response.data?._server_messages || JSON.stringify(err.response.data);
                 } else {
                    errorMsg = err.message;
                 }
                 showAlert(`Error: ${errorMsg}`, 'danger');
            } finally {
                disableSubmitButton(false);
            }
        }

        async function editStudent(name) {
             // Scroll to top to make form visible
            window.scrollTo({ top: 0, behavior: 'smooth' });
            try {
                const res = await axios.get(`${API_BASE_URL}/${name}`);
                const s = res.data.data;
                if (s) {
                    studentIdInput.value = s.student_id || '';
                    fullNameInput.value = s.full_name || '';
                    genderInput.value = s.gender || '';
                    dobInput.value = s.date_of_birth || '';
                    enrollmentDateInput.value = s.enrollment_date || '';
                    phoneInput.value = s.phone_number || '';
                    emailInput.value = s.email || '';
                    studentNameInput.value = s.name; // Store the docname for PUT request

                    setFormState(true, s.name); // Set form to edit mode
                } else {
                     throw new Error("Student data not found in response.");
                }
            } catch (err) {
                 console.error(`Error fetching student ${name} for edit:`, err);
                 const errorMsg = err.response?.data?.message || err.message || 'Failed to load student data for editing.';
                 showAlert(`Error: ${errorMsg}`, 'danger');
            }
        }

        function deleteStudent(name) {
            Swal.fire({
                title: 'Are you sure?',
                text: "You won't be able to revert this!",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#d33',
                cancelButtonColor: '#3085d6',
                confirmButtonText: 'Yes, delete it!'
            }).then(async (result) => {
                if (result.isConfirmed) {
                    try {
                        await axios.delete(`${API_BASE_URL}/${name}`);
                        Swal.fire(
                            'Deleted!',
                            'The student record has been deleted.',
                            'success'
                        );
                        // If the deleted student was being edited, clear the form
                        if (studentNameInput.value === name) {
                           clearForm();
                        }
                        loadStudents(); // Refresh list
                    } catch (err) {
                        console.error(`Error deleting student ${name}:`, err);
                        const errorMsg = err.response?.data?.message || err.message || 'Failed to delete student.';
                        Swal.fire(
                            'Error!',
                            `Could not delete student. ${errorMsg}`,
                            'error'
                        );
                    }
                }
            });
        }

        // --- Event Listeners ---
        studentForm.addEventListener('submit', handleFormSubmit);
        clearButton.addEventListener('click', clearForm);

        // --- Initial Load ---
        document.addEventListener('DOMContentLoaded', loadStudents);

    </script>
</body>
</html>