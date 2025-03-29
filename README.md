<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo"></a></p>

# Crud with image upload example in laravel 11.

- CRUD stands for Create, Read, Update, and Delete, and it is a term used to describe the four basic operations that can be performed on data.
- CRUD is commonly used in computer programming, particularly in relation to databases and web applications, to refer to the fundamental actions that can be performed on a database or other data store.

- We will create a product CRUD application using Laravel 11 in this example.
- We will create a products table with name, detail and image columns using Laravel 11 migration. 
- Then, we will create routes, a controller, views, and model files for the product module. 
- We will use Bootstrap 5 for design. Then we will create "images" folder in public folder to store images.

- You can clone or download the zipfile after that extracted and paste the pest the folder you want.
- Make the .env file through .env.example
- Run the following commands

````
composer update
````
````
php artisan migrate
````
````
php artisan serve
````

**If you want to install then follow some steps which your help to understand the laravel implementation process**

- So, let's follow the steps below to create CRUD operations with Laravel 11.

- ✅ Step 1: Install Laravel 11
- ✅ Step 2: MySQL Database Configuration
- ✅ Step 3: Create Migration
- ✅ Step 4: Create Controller and Model
- ✅ Step 5: Add Resource Route
- ✅ Step 6: Add Blade Files
- ✅ Run Laravel App

- ✅ Step 1, Run command and get clean fresh laravel new application.

````
composer create-project "laravel/laravel:^11.0" crud-app
````
- ✅ Step 2, In Laravel 11, there is a default database connection using <b>SQLite</b> but if we want to use MySQL instead, we need to add a MySQL connection with the database name, username, and password to the <b>.env file</b>

````
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=lara11_image_crud
DB_USERNAME=root
DB_PASSWORD=
````

- ✅ Step 3, Now we are going to create crud application for product. so we have to create migration for "products" table using Laravel 11 php artisan command, so first fire command:

````
php artisan make:model product -mcr
````

- Afetr fired this command three file created Product migration, Product Model, ProductControoler. you will find file in the following path.

- database\migrations\2025_03_29_153256_create_products_table.php

````
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('detail');
            $table->string('image');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('products');
    }
};
````

- ✅ Step 4, Now you will find <b>app/Models/Product.php</b> and put the below content in Product.php file:

````
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class product extends Model
{
    use HasFactory;
  
    protected $fillable = [
        'name', 'detail', 'image'
    ];
}

````


- Now you have to run this migration by following command:

````
php artisan migrate
````

- In this controller will create seven methods by default as bellow methods:
- 1) index()  2)create()  3)store()  4)show()  5)edit()  6)update()  7)destroy()
- So, let's copy bellow code and put on ProductController.php file.
- app/Http/Controllers/ProductController.php

````
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Illuminate\View\View;

class ProductController extends Controller
{
/**
* Display a listing of the resource.
* @return response()
*/
public function index(): View
{
    $products = Product::latest()->paginate(5);

    return view('products.index',compact('products'))
    ->with('i', (request()->input('page', 1) - 1) * 5);
}

/**
* Show the form for creating a new resource.
*/
public function create(): View
{
    return view('products.create');
}

/**
* Store a newly created resource in storage.
*/
public function store(Request $request): RedirectResponse
{
    $request->validate([
        'name' => 'required',
        'detail' => 'required',
        'image' => 'required|image|mimes:jpeg,png,jpg,gif,svg|max:2048',
    ]);

    $input = $request->all();

    
    if ($image = $request->file('image')) {
        $destinationPath = 'images/';
        $profileImage = date('YmdHis') . "." . $image->getClientOriginalExtension();
        $image->move($destinationPath, $profileImage);
        $input['image'] = "$profileImage";
    }

    Product::create($input);

    return redirect()->route('products.index')
    ->with('success', 'Product created successfully.');
}

/**
* Display the specified resource.
*/
public function show(Product $product): View
{
    return view('products.show', compact('product'));
}

/**
* Show the form for editing the specified resource.
*/
public function edit(Product $product): View
{
    return view('products.edit', compact('product'));
}

/**
* Update the specified resource in storage.
*/
public function update(Request $request, Product $product): RedirectResponse
{
    $request->validate([
        'name' => 'required',
        'detail' => 'required'
    ]);
    $oldFile = 'images/'.$request->oldFile;
    $input = $request->all();

    // Delete old file if it exists
        if ($image = $request->file('image')!='') {
            unlink($oldFile);
        }

    if ($image = $request->file('image')) {
        $destinationPath = 'images/';
        $profileImage = date('YmdHis') . "." . $image->getClientOriginalExtension();
        $image->move($destinationPath, $profileImage);
        $input['image'] = "$profileImage";
    }else{
        unset($input['image']);
    }

    $product->update($input);

    return redirect()->route('products.index')
    ->with('success', 'Product updated successfully');
}

/**
* Remove the specified resource from storage.
*/
public function destroy(Product $product): RedirectResponse
{
    //dd($product->image);
    $oldFile = 'images/'.$product->image;
    unlink($oldFile);
    $product->delete();

    return redirect()->route('products.index')
    ->with('success', 'Product deleted successfully');
}
}

````
- ✅ Step 5, Now we need to add a resource route for the product CRUD application. So, open your <b>routes/web.php</b> file and add the following route.

````
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ProductController;

Route::get('/', function () {
    return view('welcome');
});

    
Route::resource('products', ProductController::class);
````

- ✅ Step 5, In the last step, we have to create blade files. So mainly, we have to create a layout file and then create a new folder "products", then create blade files of the CRUD app. So finally, you have to create the following blade files:

- 1) layout.blade.php 2) index.blade.php 3) create.blade.php 4) edit.blade.php 5) show.blade.php

- resources/views/products/layout.blade.php

````
<!DOCTYPE html>
<html>
<head>
    <title>Laravel 11 CRUD with Image Upload Tutorial - Amk Dvelopment</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css" />
</head>
<body>
      
<div class="container">
    @yield('content')
</div>
      
</body>
</html>
````

- resources\views\products\index.blade.php

````
@extends('products.layout')
     
@section('content')
<div class="card mt-5">
  <h2 class="card-header">Laravel 11 CRUD with Image Upload Tutorial - Amk Development</h2>
  <div class="card-body">
          
        @session('success')
            <div class="alert alert-success" role="alert"> {{ $value }} </div>
        @endsession
  
        <div class="d-grid gap-2 d-md-flex justify-content-md-end">
            <a class="btn btn-success btn-sm" href="{{ route('products.create') }}"> <i class="fa fa-plus"></i> Create New Product</a>
        </div>
  
        <table class="table table-bordered table-striped mt-4">
            <thead>
                <tr>
                    <th width="80px">No</th>
                    <th>Image</th>
                    <th>Name</th>
                    <th>Details</th>
                    <th width="250px">Action</th>
                </tr>
            </thead>
  
            <tbody>
            @forelse ($products as $product)
                <tr>
                    <td>{{ ++$i }}</td>
                    <td><img src="/images/{{ $product->image }}" width="100px"></td>
                    <td>{{ $product->name }}</td>
                    <td>{{ $product->detail }}</td>
                    <td>
                        <form action="{{ route('products.destroy',$product->id) }}" method="POST">
             
                            <a class="btn btn-info btn-sm" href="{{ route('products.show',$product->id) }}"><i class="fa-solid fa-list"></i> Show</a>
              
                            <a class="btn btn-primary btn-sm" href="{{ route('products.edit',$product->id) }}"><i class="fa-solid fa-pen-to-square"></i> Edit</a>
             
                            @csrf
                            @method('DELETE')
                
                            <button type="submit" class="btn btn-danger btn-sm" onclick="return confirm('Are you sure?');"><i class="fa-solid fa-trash"></i> Delete</button>
                        </form>
                    </td>
                </tr>
            @empty
                <tr>
                    <td colspan="5">There are no data.</td>
                </tr>
            @endforelse
            </tbody>
  
        </table>
        
        {!! $products->withQueryString()->links('pagination::bootstrap-5') !!}
  
  </div>
</div>      
@endsection
````

- resources\views\products\edit.blade.php

````
@extends('products.layout')
     
@section('content')
<div class="card mt-5">
  <h2 class="card-header">Edit Product</h2>
  <div class="card-body">
  
    <div class="d-grid gap-2 d-md-flex justify-content-md-end">
        <a class="btn btn-primary btn-sm" href="{{ route('products.index') }}"><i class="fa fa-arrow-left"></i> Back</a>
    </div>
  
    <form action="{{ route('products.update',$product->id) }}" method="POST" enctype="multipart/form-data">
        @csrf
        @method('PUT')
  
        <input type="hidden" name="oldFile" value="{{ $product->image }}">
        <div class="mb-3">
            <label for="inputName" class="form-label"><strong>Name:</strong></label>
            <input 
                type="text" 
                name="name" 
                value="{{ $product->name }}"
                class="form-control @error('name') is-invalid @enderror" 
                id="inputName" 
                placeholder="Name">
            @error('name')
                <div class="form-text text-danger">{{ $message }}</div>
            @enderror
        </div>
  
        <div class="mb-3">
            <label for="inputDetail" class="form-label"><strong>Detail:</strong></label>
            <textarea 
                class="form-control @error('detail') is-invalid @enderror" 
                style="height:150px" 
                name="detail" 
                id="inputDetail" 
                placeholder="Detail">{{ $product->detail }}</textarea>
            @error('detail')
                <div class="form-text text-danger">{{ $message }}</div>
            @enderror
        </div>

        <div class="mb-3">
            <label for="inputImage" class="form-label"><strong>Image:</strong></label>
            <input 
                type="file" 
                name="image" 
                class="form-control @error('image') is-invalid @enderror" 
                id="inputImage">
            <img src="/images/{{ $product->image }}" width="300px">
            @error('image')
                <div class="form-text text-danger">{{ $message }}</div>
            @enderror
        </div>

        <button type="submit" class="btn btn-success"><i class="fa-solid fa-floppy-disk"></i> Update</button>
    </form>
  
  </div>
</div>
@endsection
````

- resources/views/products/show.blade.php

````
@extends('products.layout')
   
@section('content')
<div class="card mt-5">
  <h2 class="card-header">Show Product</h2>
  <div class="card-body">
  
    <div class="d-grid gap-2 d-md-flex justify-content-md-end">
        <a class="btn btn-primary btn-sm" href="{{ route('products.index') }}"><i class="fa fa-arrow-left"></i> Back</a>
    </div>
  
    <div class="row">
        <div class="col-xs-12 col-sm-12 col-md-12">
            <div class="form-group">
                <strong>Name:</strong> <br/>
                {{ $product->name }}
            </div>
        </div>
        <div class="col-xs-12 col-sm-12 col-md-12 mt-2">
            <div class="form-group">
                <strong>Details:</strong> <br/>
                {{ $product->detail }}
            </div>
        </div>
        <div class="col-xs-12 col-sm-12 col-md-12">
            <div class="form-group">
                <strong>Image:</strong><br/>
                <img src="/images/{{ $product->image }}" width="500px">
            </div>
        </div>
    </div>
  
  </div>
</div>
@endsection
````
- Make sure, you have created "images" folder in public directory
- All the required steps have been done, now you have to type the given below command and hit enter to run the Laravel app:

````
php artisan serve
````

- Now, Go to your web browser, and type the given URL and view the app output: 
- http://localhost:8000/products

- ✅ Thanks For watching if you Understand my explanation then support me and my youtube channel [AMK DEVELOPMENT](https://www.youtube.com/@amkdevelopment?sub_confirmation=1)

```
Email:    mohibulkhan15992@gmail.com
Contact:  +917007192298
````




