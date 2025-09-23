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