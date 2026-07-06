# Laravel 10 Google Login & Registration (Social Authentication)

This project demonstrates how to implement **Google OAuth Login & Registration** in **Laravel 10** using **Laravel Socialite**. It allows users to log in or register using their Google account instead of manually entering credentials.

---

## ❓ Why Use Google Social Login?

Google authentication simplifies user onboarding and improves security.

It is useful when:

- You want faster user registration without forms.
- You want to reduce password-related issues (forgot password, weak passwords).
- You want secure OAuth-based authentication.
- You want a better user experience with one-click login.

Instead of creating an account manually, users can simply sign in using their Google account.

---

## 🧩 What This Project Contains

- ✅ Manual Registration System
- ✅ Manual Login System
- ✅ Google Login (OAuth)
- ✅ Google Registration Flow
- ✅ Socialite Integration
- ✅ Duplicate Email Handling
- ✅ Automatic User Creation from Google Account
- ✅ Dashboard Protected by Auth Middleware
- ✅ Secure Logout System

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| Laravel 10 | Backend Framework |
| Laravel Socialite | Google OAuth Authentication |
| Blade | Frontend Views |
| Eloquent ORM | Database Handling |
| Bootstrap (optional) | UI Styling |

---

# 🚀 Important Steps

## 1️⃣ Install Laravel Socialite

Install the required package:

```bash
composer require laravel/socialite
```

---

## 2️⃣ Add Google Credentials in `.env`

```env
GOOGLE_CLIENT_ID=your-client-id
GOOGLE_CLIENT_SECRET=your-client-secret
GOOGLE_REDIRECT_URI=http://your-app-url/auth/google/callback
```

---

## 3️⃣ Configure Services

In `config/services.php`:

```php
'google' => [
    'client_id' => env('GOOGLE_CLIENT_ID'),
    'client_secret' => env('GOOGLE_CLIENT_SECRET'),
    'redirect' => env('GOOGLE_REDIRECT_URI'),
],
```

---

## 4️⃣ Define Routes

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Route;
use Illuminate\Http\Request;
use App\Http\Controllers\Auth\LoginController;
use App\Http\Controllers\Auth\RegisterController;

/*
|--------------------------------------------------------------------------
| Web Routes
|--------------------------------------------------------------------------
*/

// Login & Register Views
Route::get('/login', function () {
    return view('auth.login');
})->name('login');

Route::get('/register', function () {
    return view('auth.register');
})->name('register');

// Manual Registration
Route::post('/register', [RegisterController::class, 'register'])->name('register.post');

// Manual Login
Route::post('/login', function (Request $request) {

    $credentials = $request->validate([
        'email' => 'required|email',
        'password' => 'required',
    ]);

    if (Auth::attempt($credentials)) {
        $request->session()->regenerate();
        return redirect('/dashboard');
    }

    return back()->withErrors([
        'email' => 'Invalid credentials',
    ]);
})->name('login.post');

// Dashboard (Protected)
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware('auth')->name('dashboard');

// Logout
Route::get('/logout', function () {
    Auth::logout();
    return redirect('/login');
})->name('logout');

/*
|--------------------------------------------------------------------------
| Google OAuth Routes
|--------------------------------------------------------------------------
*/

// Google Login
Route::get('auth/google', [LoginController::class, 'redirectToGoogle'])->name('google.redirect');
Route::get('auth/google/callback', [LoginController::class, 'handleGoogleCallback'])->name('google.callback');

// Google Registration
Route::get('register/google', [RegisterController::class, 'redirectToGoogleRegister'])->name('register.google');
Route::get('register/google/callback', [RegisterController::class, 'handleGoogleRegisterCallback'])->name('register.google.callback');

// Google Registration Success Page
Route::get('/google-register-success', function () {
    return view('auth.google-success');
})->name('google.register.success');
```

---

## 5️⃣ Google Login Controller

```php
namespace App\Http\Controllers\Auth;

use App\Models\User;
use Illuminate\Support\Str;
use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;
use Laravel\Socialite\Facades\Socialite;

class LoginController extends Controller
{
    public function redirectToGoogle()
    {
        return Socialite::driver('google')
            ->with([
                'prompt' => 'select_account consent'
            ])
            ->redirect();
    }

    public function handleGoogleCallback()
    {
        $googleUser = Socialite::driver('google')->stateless()->user();

        $user = User::where('email', $googleUser->getEmail())->first();

        if (!$user) {
            $user = User::create([
                'name' => $googleUser->getName(),
                'email' => $googleUser->getEmail(),
                'password' => Hash::make(Str::random(16)),
                'google_id' => $googleUser->getId(),
            ]);
        }

        Auth::login($user);

        return redirect('/dashboard');
    }
}
```

---

## 6️⃣ Google Registration Controller

```php
namespace App\Http\Controllers\Auth;

use App\Models\User;
use Illuminate\Support\Str;
use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;
use Illuminate\Http\Request;
use Laravel\Socialite\Facades\Socialite;

class RegisterController extends Controller
{
    public function redirectToGoogleRegister()
    {
        return Socialite::driver('google')
            ->with([
                'prompt' => 'select_account consent'
            ])
            ->redirect();
    }

    public function handleGoogleRegisterCallback()
    {
        $googleUser = Socialite::driver('google')->stateless()->user();

        $existingUser = User::where('email', $googleUser->getEmail())->first();

        if ($existingUser) {
            return redirect()->route('login')
                ->with('error', 'Email already registered. Please login.');
        }

        User::create([
            'name' => $googleUser->getName(),
            'email' => $googleUser->getEmail(),
            'password' => Hash::make(Str::random(16)),
            'google_id' => $googleUser->getId(),
        ]);

        return redirect()->route('login')
            ->with('status', 'Registration successful. Please login.');
    }

    public function register(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:8|confirmed',
        ]);

        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),
        ]);

        Auth::login($user);

        return redirect('/dashboard');
    }
}
```

---

## 🔄 Authentication Flow

```text
Manual Flow:
Register → Login → Dashboard

Google Flow:
Google Login → OAuth Consent → Callback → Auto Login → Dashboard

Google Register Flow:
Google Register → OAuth Consent → Callback → Create Account → Login Page
```
