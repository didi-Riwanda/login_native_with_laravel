Tata cara form login di laravel :

    1. Buat route di web.php :
            code :      

                    Route::get('/login', [AuthController::class, 'showFormLogin'])->name('login');
                    Route::post('/login', [AuthController::class, 'login']);
                    Route::post('logout', [AuthController::class, 'logout'])->name('logout');

    2. Buat method sesuai route di web.php di AuthController

            code :

                    public function showFormLogin() {
                        return view('auth.login');
                    }

                    public function login(Request $request) {
                        $credentials = $request->validate([
                            'email' => ['required', 'email'],
                            'password' => ['required'],
                        ]);

                        if(Auth::attempt($credentials)) {
                            $request->session()->regenerate();
                            $user = Auth::user();

                            if ($user->role === 'admin') {
                                return 'admin';
                            } else if ($user->role === 'user') {
                                return 'user';
                            } else {
                                return 'error';
                            }
                        }

                        return back()->withErrors([
                            'email' => 'The provided credentials do not match our records.',
                        ])->onlyInput('email');
                    }

                    public function logout(Request $request) {
                        Auth::logout();

                        $request->session()->invalidate();
                        $request->session()->regenerateToken();

                        return redirect()->route('login');
                    }

    3. Buat field role di database :
            code :

                    $table->string('role')->default('user');

Tata cara buat api login dari laravel native :

    1. install laravel sactum :    
            code :

                    composer require laravel/sanctum

    2. Publish dan migrate :

            code :
                    - php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
                    - php artisan migrate:fresh --seed

    3. Tambah di model User.php HasApiTokens

            code :

                    <?php

                        namespace App\Models;
                        
                        use Laravel\Sanctum\HasApiTokens;
                        use Illuminate\Notifications\Notifiable;
                        use Illuminate\Database\Eloquent\Factories\HasFactory;
                        use Illuminate\Foundation\Auth\User as Authenticatable;
                        
                        class User extends Authenticatable
                        {
                            /** @use HasFactory<\Database\Factories\UserFactory> */
                            use HasFactory, Notifiable, HasApiTokens;
                        }

    4. Buat Api/AuthController.php untuk handle khusus API

            code :

                    <?php

                        namespace App\Http\Controllers\Api;
                        
                        use Illuminate\Http\Request;
                        use App\Http\Controllers\Controller;
                        use Illuminate\Support\Facades\Auth;
                        
                        class AuthController extends Controller
                        {
                            public function login(Request $request)
                            {
                                $credentials = $request->validate([
                                    'email' => 'required|email',
                                    'password' => 'required',
                                ]);
                        
                                if (!Auth::attempt($credentials)) {
                                    return response()->json([
                                        'message' => 'Email atau password salah'
                                    ], 401);
                                }
                        
                                $user = Auth::user();
                                $token = $user->createToken('auth_token')->plainTextToken;
                        
                                return response()->json([
                                    'access_token' => $token,
                                    'token_type' => 'Bearer',
                                    'user' => $user,
                                ]);
                            }
                        
                            public function logout(Request $request)
                            {
                                $request->user()->currentAccessToken()->delete();
                        
                                return response()->json([
                                    'message' => 'Logged out'
                                ]);
                            }
                        }

    5. Buat file route/api.php :

            code :

                    <?php

                        use Illuminate\Support\Facades\Route;
                        use App\Http\Controllers\Api\AuthController;
                        
                        Route::post('/login', [AuthController::class, 'login']);
                        Route::middleware('auth:sanctum')->post('/logout', [AuthController::class, 'logout']);
                        Route::middleware('auth:sanctum')->get('/user', fn (\Illuminate\Http\Request $request) => $request->user());

    6. Daftar kan api.php dan sactum tadi di bootstrap/app.php

            code :

                    <?php
                        
                        use Illuminate\Foundation\Application;
                        use Illuminate\Foundation\Configuration\Exceptions;
                        use Illuminate\Foundation\Configuration\Middleware;
                        use Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful;
                        
                        return Application::configure(basePath: dirname(__DIR__))
                            ->withRouting(
                                web: __DIR__.'/../routes/web.php',
                                api: __DIR__.'/../routes/api.php', // ini daftarkan
                                commands: __DIR__.'/../routes/console.php',
                                health: '/up',
                            )
                            ->withMiddleware(function (Middleware $middleware): void {
                                $middleware->alias([
                                    'auth:sanctum' => EnsureFrontendRequestsAreStateful::class, // ini juga daftarkan
                                ]);
                            })
                            ->withExceptions(function (Exceptions $exceptions): void {
                                //
                            })->create();

    7. Di config/auth.php tambahkan guard untuk api, karena sactum perlu itu 

            code :

                    'api' => [
                        'driver' => 'sanctum',
                        'provider' => 'users',
                    ],

    8. Kalau mau testing di postman :

            a. Post
                - ketik 'http://localhost:8000/api/login'
                - method nya POST
                - di tab body, pilih raw->json
                        code :
    
                            {
                              "email": "labadie.emelie@example.org",
                              "password": "password"
                            }

            b. Get
                - ketik 'http://localhost:8000/api/user'
                - method nya GET
                - di tab Authorization, pilih Auth Type nya Bearer Token
                - masukkan token dari method post yang berhasil tadi, 
                    - access_token": "1|D2ksWA67LyQEf15p1xFxvQaqKY4cRsdtTn0Un24ya8de4c60",
