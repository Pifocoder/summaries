Способы взаимодействия с Engine:
1) Обмен сообщениями с платформой
2) Встроивание нативных виджетов(например карты)
3) Подключение C/C++/... библиотек
Шаги создания federated plugin:
1) создать пакеты
2) plugin_platform_interface
3) реализация
4) подправляем pubspec.yaml
### Каналы
method chanels  - используется в большинстве случаев, когда вызов метода во Flutter запускает нативный метод. Поддерживает асинхронные вызовы методов.
```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
class CartPage extends StatelessWidget {  
	static const MethodChannel _channel = MethodChannel('co.wawand/stripe'); 
	
	@override  Widget build(BuildContext context) {    
		return Scaffold(     
		 // Widget...    
		); 
	}
}
```
event channel - для передачи потоков данных из нативного кода во Flutter. Возваращает Stream, а не future.
```dart
class TimeHandler : EventChannel.StreamHandler {
	override fun onListen(p0: Any?, sink: EventChannel.EventSink) {
    }
	override fun onCancel(p0: Any?) {
	}
 }

val eventChannel = EventChannel(flutterEngine.dartExecutor.binaryMessenger, "timeHandlerEvent"); // timeHandlerEvent event name
eventChannel.setStreamHandler(TimeHandler) // TimeHandler is event class
```
Пакет Pigeon - пакет для кодогенерации работы с Platform Channels

Отрисовка:
1) VirtualDisplay - картинка, очень быстро, но есть баги
2) Hybrid Composition - отрисовка на невидимом view(нативное view + flutter view)(меньше багов, но медленнее)(Начиная с 19 Android)
3) Texture lauer HC  - что-то новое

FFI - это **foreign function interface, или же интерфейс внешних функций**. Это механизм, позволяющий программе на одном языке вызывать функции, написанные на другом. В случае с Flutter-ом, это возможность вызывать из Dart функции Си из скомпилированных библиотек.
Есть ffigen
### [Семинар](https://github.com/nikberX/flutter_platform_communication_example)
1) Хотим настроить нативный код в приложении(перемножение матриц), то есть перемножать на самой платформе.
Как это работает кратко:
В коде приложения (main) мы описываем как работать с нативом:
```dart
// native benchmark  
stopwatch.reset();  
stopwatch.start();  
final nativeBench = await MatrixMultiplyer().multiplyMatrices(dimensions);  
stopwatch.stop();  
nativeTotal = stopwatch.elapsedMilliseconds;  
setState(() {  
  nativeResultMs = nativeBench;  
});
```
Есть пакеты, в которых мы описываем натив и interface,которы мы как раз вызывали выше для работы с этом нативом:
![](https://i.imgur.com/taOrp3G.png)
interface
```dart
import 'package:plugin_platform_interface/plugin_platform_interface.dart';  
  
abstract class MatrixMultiplyerPlatform extends PlatformInterface {  
  MatrixMultiplyerPlatform() : super(token: _token);  
  
  static final Object _token = Object();  
  
  static MatrixMultiplyerPlatform _instance = _PlaceholderImplementation();  
  
  static MatrixMultiplyerPlatform get instance => _instance;  
  
  static set instance(MatrixMultiplyerPlatform instance) {  
    PlatformInterface.verifyToken(instance, _token);  
    _instance = instance;  
  }  
  
  /// Перемножает квадратные матрицы со случайными целыми числами размера dimensions  
  ///  /// Возвращает количество миллисекунд, равное продолжительности исполнения кода перемножения  Future<int> multiplyMatrices(int dimensions) => throw UnimplementedError(  
        'MatrixMultiplyerPlatform has not been implemented',  
      );  
  
  /// Вызывает result.error на стороне натива  
  Future<int> multiplyMatricesError(int dimensions) => throw UnimplementedError(  
        'MatrixMultiplyerPlatform has not been implemented',  
      );  
  
  /// Периодически возвращает инерементируемые значения  
  Stream<int> timerStream() => throw UnimplementedError(  
        'MatrixMultiplyerPlatform has not been implemented',  
      );  
}  
  
class _PlaceholderImplementation extends MatrixMultiplyerPlatform {}
```
Сам matrix_multiplyer:
```dart
library matrix_multiplyer;  
  
import 'package:matrix_multiplyer_platform_interface/matrix_multiplyer_platform_interface.dart';  
  
class MatrixMultiplyer {  
  static MatrixMultiplyerPlatform get _platform =>  
      MatrixMultiplyerPlatform.instance;  
  
  Future<int> multiplyMatrices(int dimensions) =>  
      _platform.multiplyMatrices(dimensions);  
  
  Stream<int> timerStream() => _platform.timerStream();  
}
```
Сторона dart в matrix_multiplyer_android:
```dart
import 'package:flutter/services.dart';  
import 'package:matrix_multiplyer_platform_interface/matrix_multiplyer_platform_interface.dart';  
  
class MatrixMultiplyerAndroid extends MatrixMultiplyerPlatform {  
  static const MethodChannel _methodChannel =  
      MethodChannel('matrix_multiplyer_android');  
  
  static const EventChannel _eventChannel =  
      EventChannel('matrix_multiplyer_android/events');  
  
  /// При повторном вызове receiveBroadcastStream мы получим отдельный стрим  
  /// При закрытии из разных мест выстрелит ошибка  /// ```dart  /// PlatformException: PlatformException(error, No active stream to cancel, null, null)  ///  at StandardMethodCodec.decodeEnvelope(package:flutter/src/services/message_codecs.dart:653)  ///  at MethodChannel._invokeMethod(package:flutter/src/services/platform_channel.dart:296)  ///  at EventChannel.receiveBroadcastStream.<fn>(package:flutter/src/services/platform_channel.dart:649)  /// ```  /// В ней не будет понятного стек-трейса и ее очень сложно будет обнаружить  /// Поэтому кэшируем значение в переменную  static Stream<dynamic>? _eventChannelStream;  
  
  /// Вызывается plugin_platform_interface'ом для регистрации плагина в хостовом приложении  
  static void registerWith() {  
    MatrixMultiplyerPlatform.instance = MatrixMultiplyerAndroid();  
  }  
  
  @override  
  Future<int> multiplyMatrices(int dimensions) async {  
    final result = await _methodChannel.invokeMethod<int?>(  
      'multiplyMatrices',  
      dimensions,  
    );  
    return result ?? -1;  
  }  
  
  @override  
  Future<int> multiplyMatricesError(int dimensions) async {  
    final result = await _methodChannel.invokeMethod<int?>(  
      'multiplyMatricesError',  
      dimensions,  
    );  
    return result ?? -1;  
  }  
  
  @override  
  Stream<int> timerStream() =>  
      (_eventChannelStream ??= _eventChannel.receiveBroadcastStream())  
          .cast<int>();  
}
```
Сторона натива в matrix_multiplyer_android:
```dart
package com.example.matrix_multiplyer_android  
  
import android.os.Handler  
import android.os.Looper  
import io.flutter.embedding.engine.plugins.FlutterPlugin  
import io.flutter.plugin.common.EventChannel  
import io.flutter.plugin.common.EventChannel.EventSink  
import io.flutter.plugin.common.EventChannel.StreamHandler  
import io.flutter.plugin.common.MethodCall  
import io.flutter.plugin.common.MethodChannel  
import io.flutter.plugin.common.MethodChannel.MethodCallHandler  
import io.flutter.plugin.common.MethodChannel.Result  
import java.util.Timer  
import java.util.TimerTask  
import kotlin.random.Random  
import kotlin.system.measureNanoTime  
  
  
/** MatrixMultiplyerAndroidPlugin */  
class MatrixMultiplyerAndroidPlugin : FlutterPlugin, MethodCallHandler, StreamHandler {  
    /// Платформенный канал методов  
    private lateinit var methodChannel: MethodChannel  
  
    /// EventChannel  
    private lateinit var eventChannel: EventChannel  
  
    /// EventSink для событий из EventChannel. Их получаем на стороне dart.  
    private var eventSink: EventSink? = null  
  
    private lateinit var timer: Timer  
    private lateinit var handler: Handler  
  
    private var currentRunnable : Runnable? = null  
  
  
    var count = 0;  
  
    // overriden method from flutter plugin  
    override fun onAttachedToEngine(flutterPluginBinding: FlutterPlugin.FlutterPluginBinding) {  
        methodChannel =  
            MethodChannel(flutterPluginBinding.binaryMessenger, "matrix_multiplyer_android")  
        methodChannel.setMethodCallHandler(this)  
  
        eventChannel =  
            EventChannel(flutterPluginBinding.binaryMessenger, "matrix_multiplyer_android/events")  
        eventChannel.setStreamHandler(this)  
  
        timer = Timer()  
    }  
  
    override fun onDetachedFromEngine(binding: FlutterPlugin.FlutterPluginBinding) {  
        methodChannel.setMethodCallHandler(null)  
        eventChannel.setStreamHandler(this)  
    }  
  
    override fun onMethodCall(call: MethodCall, result: Result) {  
        when (call.method) {  
            "multiplyMatrices" -> handleMultiplyMatrices(call, result)  
            "multiplyMatricesError" -> handleMultiplyMatricesError(call, result)  
            else -> {  
                // Обязательно падаем в notImplemented по-умолчанию если название метода не совпало  
                // Иначе будем бесконечно висеть на await на стороне dart                // Постоянные висящие await = утечка памяти                result.notImplemented()  
            }  
        }  
    }  
  
    // Вызывается когда есть слушатели на стороне dart  
    override fun onListen(arguments: Any?, eventSink: EventSink?) {  
        this.eventSink = eventSink  
  
        handler = Handler(Looper.getMainLooper())  
        currentRunnable = object : Runnable {  
            override fun run() {  
                eventSink?.success(count++)  
  
                val runnable = currentRunnable  
                if (runnable != null) {  
                    handler.postDelayed(runnable, 3000)  
                }  
  
            }  
        }  
        handler.postDelayed(currentRunnable as Runnable, 3000)  
    }  
  
    // Вызывается когда сторона dart перестает слушать EventChannel  
    override fun onCancel(arguments: Any?) {  
        eventSink = null  
  
        timer.cancel()  
    }  
  
    fun sendTick() {  
        eventSink?.success(count++)  
        currentRunnable = null  
    }  
  
    fun handleMultiplyMatrices(call: MethodCall, result: Result) {  
        val dimensions = call.arguments<Int?>() as Int  
        val multiplyBench = multiplyMatricesCall(dimensions)  
        result.success(multiplyBench)  
    }  
  
    fun handleMultiplyMatricesError(call: MethodCall, result: Result) {  
        result.error("callError", "Called Error Method", "e.g. stacktrace")  
    }  
  
  
    fun multiplyMatricesCall(dimensions: Int): Long {  
        var matrix1 = Array(dimensions) { IntArray(dimensions) { 0 } }  
        var matrix2 = Array(dimensions) { IntArray(dimensions) { 0 } }  
  
        fillMatrix(matrix1, dimensions)  
        fillMatrix(matrix2, dimensions)  
  
        val time = measureNanoTime {  
            val result = multiplyMatrices(matrix1, matrix2)  
        }  
  
        return time / 1_000_000  
    }  
  
    fun fillMatrix(matrix: Array<IntArray>, n: Int) {  
        for (i in 0 until n) {  
            for (j in 0 until n) {  
                matrix[i][j] = Random.nextInt(0, 1000)  
            }  
        }  
    }  
  
    fun multiplyMatrices(matrix1: Array<IntArray>, matrix2: Array<IntArray>): Array<IntArray> {  
        val n = matrix1.size  
        val result = Array(n) { IntArray(n) }  
  
        for (i in 0 until n) {  
            for (j in 0 until n) {  
                var sum = 0  
                for (k in 0 until n) {  
                    sum += matrix1[i][k] * matrix2[k][j]  
                }  
                result[i][j] = sum  
            }  
        }  
  
        return result  
    }  
}
```
>[!info]
>Чтобы отдебажить нативный код, нужно использовать нативный дебаггер
>Attach Debagger in Android process

2) Хотим рисовать просто нативные view, то есть flutter потом перенесет их в flutter код
```dart
AndroidView(  
  viewType: NativeClickerView.viewType,  
  creationParamsCodec: const StandardMessageCodec(),  
  creationParams: initialText,  
  gestureRecognizers: {  //контролим жесты
    // https://api.flutter.dev/flutter/widgets/PlatformViewSurface/gestureRecognizers.html  
    Factory(  
      () => TapGestureRecognizer(),  
    ),  
  },  
);
```
Для android напишем, как это должно работать(большинство пишет за нас flutter)
```dart
package com.example.matrix_multiply_platform_view  
  
import androidx.annotation.NonNull  
  
import io.flutter.embedding.engine.plugins.FlutterPlugin  
import kotlin.system.measureNanoTime  
import io.flutter.plugin.common.MethodCall  
import io.flutter.plugin.common.MethodChannel  
import io.flutter.plugin.common.MethodChannel.MethodCallHandler  
import io.flutter.plugin.common.MethodChannel.Result  
import android.content.Context  
import android.view.View  
import android.widget.Button  
import io.flutter.plugin.common.StandardMessageCodec  
import io.flutter.plugin.platform.PlatformView  
import io.flutter.plugin.platform.PlatformViewFactory  
import kotlin.random.Random  
import kotlin.system.measureNanoTime  
  
/** MatrixMultiplyPlatformViewPlugin */  
class MatrixMultiplyPlatformViewPlugin: FlutterPlugin {  
  
  override fun onAttachedToEngine(flutterPluginBinding: FlutterPlugin.FlutterPluginBinding) {  
    flutterPluginBinding.platformViewRegistry.registerViewFactory(  
      ButtonPlatformViewFactory.TYPE,  
      ButtonPlatformViewFactory(flutterPluginBinding),  
    )  
  }  
  
  override fun onDetachedFromEngine(binding: FlutterPlugin.FlutterPluginBinding) {}  
}  
  
class ButtonPlatformViewFactory(private val flutterPluginBinding: FlutterPlugin.FlutterPluginBinding) :  
  PlatformViewFactory(StandardMessageCodec.INSTANCE) {  
    
  override fun create(context: Context?, viewId: Int, args: Any?): PlatformView {  
    return ButtonPlatformView(context!!, viewId, args as String, flutterPluginBinding)  
  }  
  
  companion object {  
    const val TYPE = "native_matrix_multiply";  
  }  
}  
  
  
class ButtonPlatformView(  
  context: Context,  
  id: Int,  
  creationParams: String,  
  flutterPluginBinding: FlutterPlugin.FlutterPluginBinding  
) : PlatformView {  
  
  var count = 0  
  
  private val button = Button(context).apply {  
    text = "Tap to start count"  
    setOnClickListener {  
      this.text = "Counted $count";  
      count++;  
    }  
  }  
  
  override fun getView(): View {  
    return button  
  }  
  
  override fun dispose() {  
  
  }  
}
```
3) FFI [example](https://github.com/nikberX/flutter_platform_communication_example/tree/master/packages/matrix_multiplyer_ffi)
Для создания _ffi plugin_ можно использовать flutter create --platforms=android,ios.... --template=plugin_ffi native_add
ffigen.yaml:
```yaml
# Run with `flutter pub run ffigen --config ffigen.yaml`.
name: MatrixMultiplyerFfiBindings
description: |
  Bindings for `src/matrix_multiplyer_ffi.h`.
  Regenerate bindings with `flutter pub run ffigen --config ffigen.yaml`.
output: 'lib/matrix_multiplyer_ffi_bindings_generated.dart'
headers:
  entry-points:
    - 'src/matrix_multiplyer_ffi.h'
  include-directives:
    - 'src/matrix_multiplyer_ffi.h'
preamble: |
  // ignore_for_file: always_specify_types
  // ignore_for_file: camel_case_types
  // ignore_for_file: non_constant_identifier_names
comments:
  style: any
  length: full
```
Потом вызываем ffigen, и он генерирует bindings для работы с этим кодом
Работа с нашем сишным кодом из flutter:
```dart
// You have generated a new plugin project without specifying the `--platforms`  
// flag. An FFI plugin project that supports no platforms is generated.  
// To add platforms, run `flutter create -t plugin_ffi --platforms <platforms> .`  
// in this directory. You can also find a detailed instruction on how to  
// add platforms in the `pubspec.yaml` at  
// https://flutter.dev/docs/development/packages-and-plugins/developing-packages#plugin-platforms.  
  
import 'dart:async';  
import 'dart:ffi';  
import 'dart:io';  
import 'dart:isolate';  
  
import 'matrix_multiplyer_ffi_bindings_generated.dart';  
  
int multiplyMatricesFfi(int dimensions) =>  
    _bindings.multiplyMatrices(dimensions);  
  
Future<int> multiplyMatricesFfiAsync(int dimensions) async {  
  final SendPort helperIsolateSendPort = await _helperIsolateSendPort;  
  final int requestId = _nextMultiplyRequestId++;  
  final _MultiplyRequest request = _MultiplyRequest(requestId, dimensions);  
  final Completer<int> completer = Completer<int>();  
  _multiplyRequests[requestId] = completer;  
  helperIsolateSendPort.send(request);  
  return completer.future;  
}  
  
const String _libName = 'matrix_multiplyer_ffi';  
  
/// The dynamic library in which the symbols for [MatrixMultiplyerFfiBindings] can be found.  
final DynamicLibrary _dylib = () {  
  if (Platform.isMacOS || Platform.isIOS) {  
    return DynamicLibrary.open('$_libName.framework/$_libName');  
  }  
  if (Platform.isAndroid || Platform.isLinux) {  
    return DynamicLibrary.open('lib$_libName.so');  
  }  
  if (Platform.isWindows) {  
    return DynamicLibrary.open('$_libName.dll');  
  }  
  throw UnsupportedError('Unknown platform: ${Platform.operatingSystem}');  
}();  
  
/// The bindings to the native functions in [_dylib].  
final MatrixMultiplyerFfiBindings _bindings =  
    MatrixMultiplyerFfiBindings(_dylib);  
  
/// A request to compute `multiply`.  
///  
/// Typically sent from one isolate to another.  
class _MultiplyRequest {  
  final int id;  
  final int dimensions;  
  
  const _MultiplyRequest(this.id, this.dimensions);  
}  
  
/// A response with the result of `multiply`.  
///  
/// Typically sent from one isolate to another.  
class _MultiplyResponse {  
  final int id;  
  final int benchmark;  
  
  const _MultiplyResponse(this.id, this.benchmark);  
}  
  
/// Counter to identify [_MultiplyRequest]s and [_MultiplyResponse]s.  
int _nextMultiplyRequestId = 0;  
  
/// Mapping from [_MultiplyRequest] `id`s to the completers corresponding to the correct future of the pending request.  
final Map<int, Completer<int>> _multiplyRequests = <int, Completer<int>>{};  
  
/// The SendPort belonging to the helper isolate.  
Future<SendPort> _helperIsolateSendPort = () async {  
  // The helper isolate is going to send us back a SendPort, which we want to  
  // wait for.  final Completer<SendPort> completer = Completer<SendPort>();  
  
  // Receive port on the main isolate to receive messages from the helper.  
  // We receive two types of messages:  // 1. A port to send messages on.  // 2. Responses to requests we sent.  final ReceivePort receivePort = ReceivePort()  
    ..listen((dynamic data) {  
      if (data is SendPort) {  
        // The helper isolate sent us the port on which we can sent it requests.  
        completer.complete(data);  
        return;      }  
      if (data is _MultiplyResponse) {  
        // The helper isolate sent us a response to a request we sent.  
        final Completer<int> completer = _multiplyRequests[data.id]!;  
        _multiplyRequests.remove(data.id);  
        completer.complete(data.benchmark);  
        return;      }  
      throw UnsupportedError('Unsupported message type: ${data.runtimeType}');  
    });  
  
  // Start the helper isolate.  
  await Isolate.spawn((SendPort sendPort) async {  
    final ReceivePort helperReceivePort = ReceivePort()  
      ..listen((dynamic data) {  
        // On the helper isolate listen to requests and respond to them.  
        if (data is _MultiplyRequest) {  
          final int result = _bindings.multiplyMatrices(data.dimensions);  
          final _MultiplyResponse response = _MultiplyResponse(data.id, result);  
          sendPort.send(response);  
          return;        }  
        throw UnsupportedError('Unsupported message type: ${data.runtimeType}');  
      });  
  
    // Send the port to the main isolate on which we can receive requests.  
    sendPort.send(helperReceivePort.sendPort);  
  }, receivePort.sendPort);  
  
  // Wait until the helper isolate has sent us back the SendPort on which we  
  // can start sending requests.  return completer.future;  
}();
```