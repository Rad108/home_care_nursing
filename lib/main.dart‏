import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:dio/dio.dart';
import 'package:get_it/get_it.dart';

final getIt = GetIt.instance;

void setupDependencies() {
  getIt.registerLazySingleton(() => FlutterSecureStorage());
  getIt.registerLazySingleton(() => Dio(BaseOptions(baseUrl: 'https://api.example.com/')));
  getIt.registerLazySingleton(() => AuthRepository(getIt<Dio>(), getIt<FlutterSecureStorage>()));
  getIt.registerFactory(() => AuthBloc(getIt<AuthRepository>()));
}

class AuthRepository {
  final Dio dio;
  final FlutterSecureStorage storage;
  AuthRepository(this.dio, this.storage);

  Future<Map<String, dynamic>> login(String email, String password) async {
    await Future.delayed(const Duration(seconds: 1));
    if (email.contains('@') && password.length >= 6) {
      final token = 'mock_token_${DateTime.now().millisecondsSinceEpoch}';
      await storage.write(key: 'auth_token', value: token);
      return {'token': token, 'name': 'Radwa User', 'id': '1'};
    }
    throw Exception('Invalid credentials');
  }

  Future<Map<String, dynamic>> register(String name, String email, String password) async {
    await Future.delayed(const Duration(seconds: 1));
    final token = 'mock_token_${DateTime.now().millisecondsSinceEpoch}';
    await storage.write(key: 'auth_token', value: token);
    return {'token': token, 'name': name, 'id': '2'};
  }

  Future<void> logout() async {
    await storage.delete(key: 'auth_token');
  }

  Future<bool> isLoggedIn() async {
    final token = await storage.read(key: 'auth_token');
    return token != null && token.isNotEmpty;
  }
}

abstract class AuthEvent {}
class CheckAuthStatusEvent extends AuthEvent {}
class LoginEvent extends AuthEvent { final String email, password; LoginEvent(this.email, this.password); }
class RegisterEvent extends AuthEvent { final String name, email, password; RegisterEvent(this.name, this.email, this.password); }
class LogoutEvent extends AuthEvent {}

abstract class AuthState {}
class AuthInitial extends AuthState {}
class AuthLoading extends AuthState {}
class AuthAuthenticated extends AuthState { final String name; AuthAuthenticated(this.name); }
class AuthUnauthenticated extends AuthState {}
class AuthError extends AuthState { final String message; AuthError(this.message); }

class AuthBloc extends Bloc<AuthEvent, AuthState> {
  final AuthRepository repo;
  AuthBloc(this.repo) : super(AuthInitial()) {
    on<CheckAuthStatusEvent>((event, emit) async {
      final isLogged = await repo.isLoggedIn();
      if (isLogged) {
        emit(AuthAuthenticated('Radwa'));
      } else {
        emit(AuthUnauthenticated());
      }
    });
    on<LoginEvent>((event, emit) async {
      emit(AuthLoading());
      try {
        final user = await repo.login(event.email, event.password);
        emit(AuthAuthenticated(user['name']));
      } catch (e) {
        emit(AuthError(e.toString()));
      }
    });
    on<RegisterEvent>((event, emit) async {
      emit(AuthLoading());
      try {
        final user = await repo.register(event.name, event.email, event.password);
        emit(AuthAuthenticated(user['name']));
      } catch (e) {
        emit(AuthError(e.toString()));
      }
    });
    on<LogoutEvent>((event, emit) async {
      await repo.logout();
      emit(AuthUnauthenticated());
    });
  }
}

enum BookingStep { initial, service, schedule, remarks, confirm, success }

class Booking {
  final String? serviceId, serviceName, remarks;
  final DateTime? scheduledAt;
  final BookingStep step;
  Booking({
    this.serviceId,
    this.serviceName,
    this.remarks,
    this.scheduledAt,
    this.step = BookingStep.initial,
  });

  Booking copyWith({
    String? serviceId,
    String? serviceName,
    String? remarks,
    DateTime? scheduledAt,
    BookingStep? step,
  }) {
    return Booking(
      serviceId: serviceId ?? this.serviceId,
      serviceName: serviceName ?? this.serviceName,
      remarks: remarks ?? this.remarks,
      scheduledAt: scheduledAt ?? this.scheduledAt,
      step: step ?? this.step,
    );
  }
}

abstract class BookingEvent {}
class SelectServiceEvent extends BookingEvent { final String id, name; SelectServiceEvent(this.id, this.name); }
class SelectScheduleEvent extends BookingEvent { final DateTime date; SelectScheduleEvent(this.date); }
class EnterRemarksEvent extends BookingEvent { final String remarks; EnterRemarksEvent(this.remarks); }
class ConfirmBookingEvent extends BookingEvent {}
class ResetBookingEvent extends BookingEvent {}

class BookingState {
  final Booking booking;
  final bool isLoading;
  final String? error;
  BookingState({required this.booking, this.isLoading = false, this.error});

  BookingState copyWith({Booking? booking, bool? isLoading, String? error}) {
    return BookingState(
      booking: booking ?? this.booking,
      isLoading: isLoading ?? this.isLoading,
      error: error ?? this.error,
    );
  }
}

class BookingBloc extends Bloc<BookingEvent, BookingState> {
  BookingBloc() : super(BookingState(booking: Booking())) {
    on<SelectServiceEvent>((event, emit) {
      emit(state.copyWith(
        booking: state.booking.copyWith(
          serviceId: event.id,
          serviceName: event.name,
          step: BookingStep.schedule,
        ),
        error: null,
      ));
    });
    on<SelectScheduleEvent>((event, emit) {
      if (event.date.isBefore(DateTime.now())) {
        emit(state.copyWith(error: 'لا يمكن اختيار موعد سابق'));
        return;
      }
      emit(state.copyWith(
        booking: state.booking.copyWith(
          scheduledAt: event.date,
          step: BookingStep.remarks,
        ),
        error: null,
      ));
    });
    on<EnterRemarksEvent>((event, emit) {
      if (event.remarks.length > 500) {
        emit(state.copyWith(error: 'الملاحظات طويلة جداً'));
        return;
      }
      emit(state.copyWith(
        booking: state.booking.copyWith(
          remarks: event.remarks,
          step: BookingStep.confirm,
        ),
        error: null,
      ));
    });
    on<ConfirmBookingEvent>((event, emit) async {
      emit(state.copyWith(isLoading: true, error: null));
      await Future.delayed(const Duration(seconds: 1));
      emit(state.copyWith(
        booking: state.booking.copyWith(step: BookingStep.success),
        isLoading: false,
      ));
    });
    on<ResetBookingEvent>((event, emit) {
      emit(BookingState(booking: Booking()));
    });
  }
}

class LoginScreen extends StatefulWidget {
  @override
  _LoginScreenState createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _emailController = TextEditingController(text: 'test@example.com');
  final _passwordController = TextEditingController(text: '123456');
  final _formKey = GlobalKey<FormState>();

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  void _login() {
    if (_formKey.currentState!.validate()) {
      context.read<AuthBloc>().add(LoginEvent(_emailController.text, _passwordController.text));
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home Care Nursing')),
      body: BlocListener<AuthBloc, AuthState>(
        listener: (context, state) {
          if (state is AuthError) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text(state.message), backgroundColor: Colors.red),
            );
          }
        },
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Form(
            key: _formKey,
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                TextFormField(
                  controller: _emailController,
                  decoration: const InputDecoration(labelText: 'Email'),
                  validator: (v) => v!.contains('@') ? null : 'أدخل بريداً صالحاً',
                ),
                const SizedBox(height: 16),
                TextFormField(
                  controller: _passwordController,
                  decoration: const InputDecoration(labelText: 'Password'),
                  obscureText: true,
                  validator: (v) => v!.length >= 6 ? null : 'كلمة المرور 6 أحرف على الأقل',
                ),
                const SizedBox(height: 24),
                BlocBuilder<AuthBloc, AuthState>(
                  builder: (context, state) {
                    if (state is AuthLoading) return const CircularProgressIndicator();
                    return ElevatedButton(onPressed: _login, child: const Text('Login'));
                  },
                ),
                TextButton(
                  onPressed: () => Navigator.push(context, MaterialPageRoute(builder: (_) => const RegisterScreen())),
                  child: const Text('Register'),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

class RegisterScreen extends StatefulWidget {
  const RegisterScreen({Key? key}) : super(key: key);
  @override
  _RegisterScreenState createState() => _RegisterScreenState();
}

class _RegisterScreenState extends State<RegisterScreen> {
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  final _passController = TextEditingController();
  final _formKey = GlobalKey<FormState>();

  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    _passController.dispose();
    super.dispose();
  }

  void _register() {
    if (_formKey.currentState!.validate()) {
      context.read<AuthBloc>().add(RegisterEvent(_nameController.text, _emailController.text, _passController.text));
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Register')),
      body: BlocListener<AuthBloc, AuthState>(
        listener: (context, state) {
          if (state is AuthError) {
            ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(state.message)));
          }
        },
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Form(
            key: _formKey,
            child: Column(
              children: [
                TextFormField(controller: _nameController, decoration: const InputDecoration(labelText: 'Name'), validator: (v) => v!.isEmpty ? 'الاسم مطلوب' : null),
                TextFormField(controller: _emailController, decoration: const InputDecoration(labelText: 'Email'), validator: (v) => v!.contains('@') ? null : 'بريد صالح'),
                TextFormField(controller: _passController, decoration: const InputDecoration(labelText: 'Password'), obscureText: true, validator: (v) => v!.length >= 6 ? null : '6 أحرف'),
                const SizedBox(height: 20),
                BlocBuilder<AuthBloc, AuthState>(
                  builder: (context, state) {
                    if (state is AuthLoading) return const CircularProgressIndicator();
                    return ElevatedButton(onPressed: _register, child: const Text('Register'));
                  },
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

class DashboardScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Dashboard'),
        actions: [IconButton(onPressed: () => context.read<AuthBloc>().add(LogoutEvent()), icon: const Icon(Icons.logout))],
      ),
      body: BlocBuilder<AuthBloc, AuthState>(
        builder: (ctx, state) {
          if (state is AuthAuthenticated) {
            return Column(children: [
              Card(child: ListTile(title: Text('Welcome ${state.name}'), subtitle: const Text('Active Bookings: 2'))),
              Expanded(
                child: ListView(children: const [
                  ListTile(title: Text('🏥 Morning Visit'), subtitle: Text('Today 10:00 AM')),
                  ListTile(title: Text('🩺 Evening Care'), subtitle: Text('Today 6:00 PM')),
                  Divider(),
                  ListTile(title: Text('📜 History: March 15, 2025'), subtitle: Text('Completed - Nurse Visit'))
                ]),
              ),
              ElevatedButton(
                onPressed: () => Navigator.push(context, MaterialPageRoute(builder: (_) => const BookingScreen())),
                child: const Text('New Booking'),
              )
            ]);
          }
          return const Center(child: Text('Not authenticated'));
        },
      ),
    );
  }
}

class BookingScreen extends StatefulWidget {
  const BookingScreen({Key? key}) : super(key: key);
  @override
  _BookingScreenState createState() => _BookingScreenState();
}

class _BookingScreenState extends State<BookingScreen> {
  late BookingBloc _bookingBloc;
  final TextEditingController _remarksController = TextEditingController();

  @override
  void initState() {
    super.initState();
    _bookingBloc = BookingBloc();
  }

  @override
  void dispose() {
    _bookingBloc.close();
    _remarksController.dispose();
    super.dispose();
  }

  void _submitRemarks() {
    if (_remarksController.text.isNotEmpty) {
      _bookingBloc.add(EnterRemarksEvent(_remarksController.text));
    } else {
      ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('الرجاء إدخال ملاحظات')));
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Book Service')),
      body: BlocProvider.value(
        value: _bookingBloc,
        child: BlocConsumer<BookingBloc, BookingState>(
          listener: (context, state) {
            if (state.error != null) {
              ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(state.error!), backgroundColor: Colors.red));
              _bookingBloc.add(ResetBookingEvent());
            }
          },
          builder: (context, state) {
            final booking = state.booking;
            if (booking.step == BookingStep.success) {
              return Center(
                child: Column(mainAxisAlignment: MainAxisAlignment.center, children: [
                  const Icon(Icons.check_circle, size: 80, color: Colors.green),
                  const Text('Booking Confirmed!', style: TextStyle(fontSize: 24)),
                  ElevatedButton(
                    onPressed: () {
                      _bookingBloc.add(ResetBookingEvent());
                      Navigator.pop(context);
                    },
                    child: const Text('Back to Dashboard'),
                  )
                ]),
              );
            }
            if (booking.step == BookingStep.initial) {
              return ListView(children: [
                ListTile(
                  title: const Text('🏠 Basic Nursing Care'),
                  onTap: () => _bookingBloc.add(SelectServiceEvent('1', 'Basic Nursing Care')),
                ),
                ListTile(
                  title: const Text('🩺 Advanced Medical Care'),
                  onTap: () => _bookingBloc.add(SelectServiceEvent('2', 'Advanced Medical Care')),
                ),
              ]);
            }
            if (booking.step == BookingStep.schedule) {
              return CalendarDatePicker(
                initialDate: DateTime.now().add(const Duration(days: 1)),
                firstDate: DateTime.now(),
                lastDate: DateTime.now().add(const Duration(days: 30)),
                onDateChanged: (date) => _bookingBloc.add(SelectScheduleEvent(date)),
              );
            }
            if (booking.step == BookingStep.remarks) {
              return Padding(
                padding: const EdgeInsets.all(16.0),
                child: Column(children: [
                  TextField(
                    controller: _remarksController,
                    maxLines: 4,
                    decoration: const InputDecoration(labelText: 'Patient Remarks', hintText: 'اذكر أي تفاصيل مهمة...'),
                  ),
                  const SizedBox(height: 20),
                  ElevatedButton(onPressed: _submitRemarks, child: const Text('Next')),
                ]),
              );
            }
            if (booking.step == BookingStep.confirm) {
              return Column(children: [
                ListTile(title: Text('Service: ${booking.serviceName}')),
                ListTile(title: Text('Date: ${booking.scheduledAt}')),
                ListTile(title: Text('Remarks: ${booking.remarks ?? "لا توجد ملاحظات"}')),
                ElevatedButton(
                  onPressed: state.isLoading ? null : () => _bookingBloc.add(ConfirmBookingEvent()),
                  child: state.isLoading ? const CircularProgressIndicator() : const Text('Confirm Booking'),
                )
              ]);
            }
            return const SizedBox();
          },
        ),
      ),
    );
  }
}

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  setupDependencies();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Home Care Nursing',
      debugShowCheckedModeBanner: false,
      home: BlocProvider(
        create: (_) => getIt<AuthBloc>()..add(CheckAuthStatusEvent()),
        child: BlocBuilder<AuthBloc, AuthState>(
          builder: (context, state) {
            if (state is AuthAuthenticated) return const DashboardScreen();
            if (state is AuthUnauthenticated) return const LoginScreen();
            if (state is AuthError) return Scaffold(body: Center(child: Text(state.message)));
            return const Scaffold(body: Center(child: CircularProgressIndicator()));
          },
        ),
      ),
    );
  }
}
