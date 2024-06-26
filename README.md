defined('AVTAR_PATH') or define('AVTAR_PATH', public_path('/backend/avtar'));
defined('AVTAR_ROOT') or define('AVTAR_ROOT', asset('/backend/avtar') . '/');


        if ($request->hasFile('avtar')) {
            $file = $request->file('avtar');
            if ($file->isValid()) {
                $ext = strtolower($file->getClientOriginalExtension());
                $filename = $client->id . '.' . $ext;
                $targetPath = AVTAR_PATH;
                $file->move($targetPath, $filename);
                $client->avtar = $filename;
                $client->save();
            } else {
                return redirect()->back()->with("avtar", "Uploaded image is not valid.");
            }
        }


           return DataTables::of($clients)
                ->addColumn('avtar', function ($row) {
                    return '<img src="' . asset('/backend/avtar/' . $row->avtar) . '" border="2" width="90" class="img-rounded" align="center" />';
                })


                 {
                        data: 'avtar',
                        name: 'avtar',
                        render: function(data, type, full, meta) {
                            return data;
                        }
                    },
<?php

namespace App\Http\Controllers;

use App\Models\Employee;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;
use Yajra\DataTables\Facades\DataTables;

class EmployeeController extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function employee()
    {
        return view('backend.employee');
    }

    /**
     * Show the form for creating a new resource.
     */
    public function employeeList()
    {
        return view('backend.employeeList');
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'name' => 'required',
            'email' => 'required',
            'phone' => 'required',
        ]);
        if ($validator->fails()) {
            return back()->withErrors($validator)->withInput();
        }
        $data = Employee::create([
            'name' => $request->name,
            'email' => $request->email,
            'phone' => $request->phone,
        ]);

        return redirect()->route('employee.view');
    }

    /**
     * Display the specified resource.
     */
    public function employeegetdata(Request $request)
    {
        if ($request->ajax()) {
            $data = Employee::select('*')->get();
            return DataTables::of($data)
                ->addColumn('action', function ($row) {
                    $editButton = '<button class="btn btn-dark edit-btn btn-sm" data-row-id="' . $row->id . '" ><i class="fa-solid fa-pencil"></i></button>&nbsp;';
                    $deleteButton = '<button class="btn btn-dark delete-btn btn-sm" data-row-id="' . $row->id . '" ><i class="fa-solid fa-delete-left"></i></button>&nbsp;';
                    return $editButton . $deleteButton;
                })

                ->rawColumns(['action'])
                ->make(true);
        }
    }

    /**
     * Show the form for editing the specified resource.
     */
    public function edit(string $id)
    {
        $user = employee::findOrfail($id);
        if ($user) {
            return view('backend.employee-edit', compact('user'));
        } else {
            return redirect()->back()->with('error', 'invalid id!');
        }
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(Request $request, string $id)
    {
        $data = $request->all();
        $user = employee::findOrfail($id);
        $rule = [
            'name' => 'required',
            'email' => 'required',
            'phone' => 'required',
        ];
        $validator = Validator::make($data, $rule);
        if ($validator->fails()) {
            return back()->withErrors($validator)->withInput();
        }
        $user->update($data);
        return redirect()->route('employeeList')->with('success', 'Employee updated successfully');
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(string $id)
    {
        $user = employee::find($id);
        $user->delete();
        return response()->json(['success' => 'employee delete successfully']);
    }
}
@extends('backend.layouts.main')
@section('title', 'MT : : Admin Profile')
@section('content')
    <div class="container-xxl flex-grow-1 container-p-y">
        <div class="row">
            <!-- Basic with Icons -->
            <div class="col-xxl">
                <div class="card mb-4">
                    <div class="card-header d-flex align-items-center justify-content-between">
                        <h5 class="mb-0">Admin Profile</h5>
                        <small class="text-muted float-end">Merged input group</small>
                    </div>
                    <div class="card-body">
                        <table class="table" id="employeetable">
                            <thead>
                                <tr>
                                    <th>Name</th>
                                    <th>Email</th>
                                    <th>Phone</th>
                                    <th>action</th>
                                </tr>
                            </thead>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </div>
@endsection

@section('script')
    <script src="https://cdn.datatables.net/1.10.19/js/jquery.dataTables.min.js"></script>
    <script>
        $(document).ready(function() {
            $('#employeetable').DataTable({
                processing: true,
                serverSide: true,
                ajax: '{{ route('employee.getdata') }}',
                columns: [{
                        data: 'name',
                        name: 'name'
                    },
                    {
                        data: 'email',
                        name: 'email'
                    },
                    {
                        data: 'phone',
                        name: 'phone'
                    }, {
                        data: 'action',
                        name: 'action',
                        orderable: false,
                        searchable: true
                    }
                ]
            });

            $(document).on('click', '.delete-btn', function(e) {
                var deleteBtn = $(this);
                var id = deleteBtn.data('row-id');
                var r = confirm('Are you sure you want to delete?');
                if (!r) {
                    return false;
                }
                $.ajax({
                    type: "POST",
                    url: "/employee-delete/" + id,
                    data: {
                        _token: "{{ csrf_token() }}",
                        _method: "DELETE"
                    },
                    success: function(resp) {
                        if (resp.success) {
                            $('#employeetable').DataTable().ajax.reload();
                            $('.alert-success .msg-content').html(resp.message);
                            $('.alert-success').show();
                        } else {
                            $('.alert-danger .msg-content').html(resp.message);
                            $('.alert-danger').show();
                        }
                        deleteBtn.attr('disabled', false);
                    },
                    error: function(xhr, status, error) {
                        alert('Error: ' + error);
                        deleteBtn.attr('disabled', false);
                    }
                });
            });

            $("#employeetable").on('click', '.edit-btn', function() {
                var editBtn = $(this);
                var id = editBtn.data('row-id'); // Change to data('row-id')
                window.location.href = "/employee-edit/" + id;
            });


        });
    </script>
@endsection
<?php

use App\Http\Controllers\AdmissionController;
use App\Http\Controllers\EmployeeController;
use App\Http\Controllers\TestController;
use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('backend.dashboard');
});


// TEST
Route::get('/testview', [TestController::class, 'testview'])->name('testview');
Route::post('/test/store', [TestController::class, 'teststore'])->name('teststore');
Route::post('/test/edit/{id}', [TestController::class, 'edit'])->name('edit');
Route::put('/test/update/{id}', [TestController::class, 'update'])->name('update');
Route::post('/test/{id}', [TestController::class, 'destroy'])->name('destroy');
Route::get('/testview/getdata', [TestController::class, 'getData'])->name('getData');


//ADMISSION
Route::get('/addmission', [AdmissionController::class, 'index'])->name('admission');
Route::post('/addmission/store', [AdmissionController::class, 'store'])->name('admission.store');
Route::get('/addmission/getdata', [AdmissionController::class, 'getAdmissiondata'])->name('getAdmissiondata');
Route::post('/addmission/{id}', [AdmissionController::class, 'destroy'])->name('admission.destroy');


Route::get('/employee-add', [EmployeeController::class, 'employee'])->name('employee.view');

Route::post('/employee-store', [EmployeeController::class, 'store'])->name('employee.store');
Route::get('/employee-List', [EmployeeController::class, 'employeeList'])->name('employeeList');
Route::get('/employee-getdata', [EmployeeController::class, 'employeegetdata'])->name('employee.getdata');
Route::get('/employee-edit/{id}', [EmployeeController::class, 'edit'])->name('employee.edit');
Route::put('/employee-update/{id}', [EmployeeController::class, 'update'])->name('employee.update');
Route::delete('/employee-delete/{id}', [EmployeeController::class, 'destroy'])->name('employee.destroy');



// LEAVING_CERTIFICATE
defined('LEAVING_CERTIFICATE') or define('LEAVING_CERTIFICATE', public_path('/backend/admission/leaving_certificate'));

//MEDICAL_REPORT
defined('MEDICAL_REPORT') or define('MEDICAL_REPORT', public_path('/backend/admission/medical_report'));

//REPORT_CARD
defined('REPORT_CARD') or define('REPORT_CARD', public_path('/backend/admission/report_card'));
@extends('backend.layouts.main')
@section('title', 'MT : : Admin Profile')
@section('content')

    <div class="container-xxl flex-grow-1 container-p-y">

        <div class="row">

            <!-- Modal -->
            <div class="modal fade" id="profileModal" tabindex="-1" aria-labelledby="profileModalLabel" aria-hidden="true">
                <div class="modal-dialog">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h5 class="modal-title" id="profileModalLabel">Add Profile</h5>
                            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                        </div>
                        <div class="modal-body">
                            <form action="{!! route('teststore') !!}" method="POST" id="myform">
                                @csrf
                                <div class="mb-3">
                                    <label for="fullname" class="form-label">Name</label>
                                    <input type="text" class="form-control" name="name" placeholder="John Doe">
                                </div>
                                <div class="mb-3">
                                    <label for="email" class="form-label">Email</label>
                                    <input type="email" class="form-control" name="email"
                                        placeholder="john.doe@example.com">
                                </div>
                                <div class="mb-3">
                                    <label for="phone" class="form-label">Phone</label>
                                    <input type="text" class="form-control" name="phone" placeholder="Phone">
                                </div>
                                <div class="modal-footer">
                                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
                                    <button type="submit" class="btn btn-primary">Save changes</button>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
            </div>



            <!-- Edit Modal -->
            <div class="modal fade" id="editProfileModal" tabindex="-1" aria-labelledby="editProfileModalLabel"
                aria-hidden="true">
                <div class="modal-dialog">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h5 class="modal-title" id="editProfileModalLabel">Edit Profile</h5>
                            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                        </div>
                        <div class="modal-body">
                            <form action="" method="POST" id="editForm">
                                @csrf
                                @method('PUT')
                                <div class="mb-3">
                                    <label for="editName" class="form-label">Name</label>
                                    <input type="text" class="form-control" id="editName" name="name"
                                        placeholder="John Doe">
                                </div>
                                <div class="mb-3">
                                    <label for="editEmail" class="form-label">Email</label>
                                    <input type="email" class="form-control" id="editEmail" name="email"
                                        placeholder="john.doe@example.com">
                                </div>
                                <div class="mb-3">
                                    <label for="editPhone" class="form-label">Phone</label>
                                    <input type="text" class="form-control" id="editPhone" name="phone"
                                        placeholder="Phone">
                                </div>
                                <div class="modal-footer">
                                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
                                    <button type="submit" class="btn btn-primary">Save changes</button>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
            </div>

            <!-- End Modal -->

            <!-- End Modal -->
            <!-- Basic with Icons -->
            <div class="col-xxl">
                <div class="card mb-4">
                    <div class="card-header d-flex align-items-center justify-content-between">
                        <h5 class="mb-0"> Profile</h5>
                        <small class="text-muted float-end">Merged input group</small>
                    </div>
                    <div class="card-body">
                        <button type="button" class="btn btn-primary" data-bs-toggle="modal"
                            data-bs-target="#profileModal">Add Profile</button>
                        <table class="table" id="getdata">
                            <thead>
                                <tr>
                                    <th>Name</th>
                                    <th>email</th>
                                    <th>phone</th>
                                    <th>action</th>
                                </tr>
                            </thead>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </div>
@endsection


@section('script')
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script src="https://cdn.datatables.net/1.11.5/js/jquery.dataTables.min.js"></script>

    <script>
        $(document).ready(function() {
            $('#myform').submit(function(e) {
                e.preventDefault();
                $.ajax({
                    type: "POST",
                    url: "{{ route('teststore') }}",
                    data: $(this).serialize(),
                    success: function(response) {
                        console.log(response);
                        $('#getdata').DataTable().ajax.reload();
                        $('#myform')[0].reset();
                        $('#profileModal').modal('hide');
                        alert('Data submitted successfully!');
                    },
                    error: function(error) {
                        console.log(error);
                        alert('Error occurred while submitting data.');
                    }
                });
            });

            $('#getdata').DataTable({
                processing: true,
                serverSide: true,
                ajax: {
                    url: "{{ route('getData') }}",
                    type: 'GET',
                },
                columns: [{
                        data: 'name',
                        name: 'name',
                        orderable: false,
                        searchable: true
                    },
                    {
                        data: 'email',
                        name: 'email',
                        orderable: false,
                        searchable: true
                    },
                    {
                        data: 'phone',
                        name: 'phone',
                        orderable: false,
                        searchable: true
                    },
                    {
                        data: 'action',
                        name: 'action',
                        orderable: false,
                        searchable: true
                    },
                ]
            });

            $(document).on('click', '.edit-btn', function() {
                var id = $(this).data('row-id');
                $.ajax({
                    type: "post",
                    url: "/test/edit/" + id,
                    data: {
                        _token: "{!! csrf_token() !!}"
                    },
                    success: function(response) {
                        $('#editName').val(response.data.name);
                        $('#editEmail').val(response.data.email);
                        $('#editPhone').val(response.data.phone);
                        $('#editForm').attr('action', '/test/' + id);
                        $('#editProfileModal').modal('show');
                    },
                    error: function(error) {
                        console.log(error);
                        alert('Error occurred while fetching data.');
                    }
                });
            });


            $('#editForm').submit(function(e) {
                e.preventDefault();
                var id = $(this).attr('action').split('/').pop();
                $.ajax({
                    type: "PUT",
                    url: "/test/update/" + id,
                    data: $(this).serialize(),
                    success: function(response) {
                        console.log(response);
                        $('#getdata').DataTable().ajax.reload();
                        $('#editProfileModal').modal('hide');
                        alert('Data Updated successfully!');
                    },
                    error: function(error) {
                        console.log(error);
                        alert('Error occurred while submitting data.');
                    }
                });
            });



            $(document).on('click', '.delete-btn', function(e) {
                var deleteBtn = $(this);
                var id = deleteBtn.data('row-id');
                var r = confirm("Are you sure to delete this?");
                if (!r) {
                    return false;
                }
                $.ajax({
                    type: "post",
                    url: "/test/" + id,
                    data: {
                        _token: "{!! csrf_token() !!}"
                    },
                    success: function(resp) {
                        if (resp.success) {
                            $('#getdata').DataTable().ajax.reload();
                            $('.alert-success .msg-content').html(resp.message);
                            $('.alert-success').show();
                        } else {
                            $('.alert-danger .msg-content').html(resp.message);
                            $('.alert-danger').show();
                        }
                        deleteBtn.attr('disabled', false);
                        table.draw();
                    },
                    error: function(e) {
                        alert('Error: ' + e);
                        deleteBtn.attr('disabled', false);
                    }
                });
            });

        });
    </script>

@endsection
<?php

namespace App\Http\Controllers;

use App\Models\test;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;
use Yajra\DataTables\Facades\DataTables;

class TestController extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function testview()
    {
        return view('backend.test');
    }

    /**
     * Show the form for creating a new resource.
     */
    public function create()
    {
        //
    }

    /**
     * Store a newly created resource in storage.
     */
    public function teststore(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'name' => "required",
            'email' => 'required',
            'phone' => "required",
        ]);

        if ($validator->fails()) {
            return response()->json(['success' => false, 'error' => $validator->errors()]);
        }

        $user = test::create([
            'name' => $request->name,
            'email' => $request->email,
            'phone' => $request->phone
        ]);
        if ($user) {
            return response()->json(['success' => true, 'message' => 'data added successfully!']);
        } else {
            return response()->json(['success' => false, 'message' => 'Failed to add user.']);
        }
    }

    public function getData(Request $request)
    {
        if ($request->ajax()) {
            $data = test::select('*')->get();
            return DataTables::of($data)

                ->addColumn('action', function ($row) {
                    $editbutton = '<button class = "btn btn-success  edit-btn btn-sm" data-row-id="' . $row->id . '"><i class="fa-solid fa-pencil"></i></button>&nbsp;';
                    $deleteButton = '<button class="btn btn-danger  delete-btn btn-sm" data-row-id="' . $row->id . '"><i class="fa-solid fa-delete-left"></i></button>&nbsp;';
                    return $editbutton . $deleteButton;
                })
                ->rawColumns(['action'])
                ->make(true);
        }
    }


    /**
     * Display the specified resource.
     */
    public function show(string $id)
    {
        //
    }

    /**
     * Show the form for editing the specified resource.
     */
    public function edit(string $id)
    {
        $user = Test::findOrFail($id);
        if ($user) {
            return response()->json(['data' => $user]);
        } else {
            return response()->json(['error' => 'Invalid ID'], 404);
        }
    }


    /**
     * Update the specified resource in storage.
     */
    public function update(Request $request, string $id)
    {
        $validator = Validator::make($request->all(), [
            'name' => "required",
            'email' => "required",
            'phone' => "required",
        ]);
        if ($validator->fails()) {
            return response()->json(['success' => false, 'error' => $validator->errors()]);
        }
        $user = test::find($id);
        if (!$user) {
            return response()->json(['message' => 'User not found.']);
        }
        $user->name = $request->name;
        $user->email = $request->email;
        $user->phone = $request->phone;
        $user->save();

        return response()->json(['success' => true, 'message' => 'profile update sucessfully']);
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(string $id)
    {
        $user = test::find($id);
        $user->delete();
        return response()->json(['success' => 'delete sucessfully!!']);
    }
}

@extends('backend.layouts.main')
@section('title', 'MT : : Admin Profile')
@section('content')

    <div class="container-xxl flex-grow-1 container-p-y">

        <div class="row">

            <div class="col-xxl">
                <div class="card mb-4">
                    <div class="card-header d-flex align-items-center justify-content-between">
                        <h5 class="mb-0">Admin Profile</h5>
                        <small class="text-muted float-end">Merged input group</small>
                    </div>
                    <div class="card-body">
                        <form id="profileForm" action="{{ route('employee.update', ['id' => $user->id]) }}" method="POST">
                            @csrf
                            @method('PUT')
                            <div class="row mb-3">
                                <input type="hidden" name="_method" value="PUT">

                                <label class="col-sm-2 col-form-label" for="name">Name</label>
                                <div class="col-sm-10">
                                    <input type="text" class="form-control" name="name" id="name"
                                        value="{{ $user->name }}" placeholder="John Doe" />
                                </div>
                            </div>

                            <div class="row mb-3">
                                <label class="col-sm-2 col-form-label" for="email">Email</label>
                                <div class="col-sm-10">
                                    <input type="text" id="email" class="form-control" name="email"
                                        value="{{ $user->email }}" placeholder="john.doe@example.com" />
                                </div>
                            </div>

                            <div class="row mb-3">
                                <label class="col-sm-2 col-form-label" for="phone">Phone</label>
                                <div class="col-sm-10">
                                    <input type="text" class="form-control" name="phone" id="phone"
                                        value="{{ $user->phone }}" placeholder="Phone Number" />
                                </div>
                            </div>
                            <div class="row justify-content-end">
                                <div class="col-sm-10">
                                    <button type="submit" class="btn btn-primary">Update</button>
                                </div>
                            </div>
                        </form>
                    </div>
                </div>
            </div>

        </div>

    </div>

@endsection

