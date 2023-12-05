---
{"dg-publish":true,"permalink":"/3-dev/flutter/riverpod/riverpod-tutorial-03/","tags":["flutter","riverpod","tutorial"]}
---


```dart
final testFutureProvider = FutureProvider<String>((ref) async {
  final dio = ref.watch(dioProdiver);
  final response = await dio.get<dynamic>("https://googld.com");
  return response.toString();
})

final response = watch(testFuntureProvider);
return respone.when(
	data: (response) => Text(response),
  	loading: () => const CircularProgressIndicator(),
  	error: (erro, st) => Text(err.toString()),
);
```

> 기본으로 설정된 곳에 페이지가 생기는구나. 

#file todo_state.dart
```dart
class TodosNotifier extends StateNotifier<AsyncValue<List<Todo>>> {
	TodosNotifier(
		this.read, [
		AsyncValue<List<Todo>> todos,			
	]) : super(todos ?? const AsyncValue.loading()){
		_retrieveTodos();
	}
	final Reader read;
	AsyncValue<List<Todo>> previousState;

	Future<void> _retrieveTodos() async {
		try {
			final todos = await read(todoRepositoryProvider).retrieveTodos();
			state = AsyncValue.data(todos);
		} on TodoException catch (e, st) {
			state = AsyncValue.error(e,st);
		}
	}
}
```
위의 #Notifier 를 사용하면, 
```dart
final todosState = watch(todosNotifierProvider.state);
return todosState.when(
	data: (todos) {}
	loading: () {}
	error: (err, stacktrace) {}
)
```

#file todo_repository.dart
```dart
abstract class TodoRepository {
	Future<List<Todo>> retrieveTodos();
	Future<void> addTodo(String description);
	Future<void> toggle(String id);
	Future<void> edit({
		@required String id,
		@required String description,
	});
	Future<void> remove(String id);
}
```


#file todo_state.dart
```dart
final todoRepositoryProvider = Provider<TodoRepository>((ref) {
	throw UnimplementedError();
})

final todosNotifierProvider = StateNotifierProvider<TodosNotifier>((ref) {
	return TodosNotifier(ref.read);
})

final todoExceptionProvider = StateProvider<TodoException>((ref) {
	return null;
})
 
...
void _cacheState() {
	previousState = state.whenData((value) => value)
}
void _resetState() {
	if (previousState != null) {
		state = previousState;
		previousState = null;
	}
}
void _handleException(TodoException e) {
	_resetState();
	read(todoExceptionProvider).state = e
}
```


main.dart 에서
```dart
body: ProviderListener(
	provider: todoExceptionProvider,
	onChange: (
		BuildContext context,
		StateController<TodoException> exceptionState,
	) {
		Scaffold.of(context).showSnackBar(
			SnackBar(
				content: Text(
					exceptionState.state.error.toString()	
				)
			)
		)
	},
	child: SafeArea(
		child: TabBarView(
		
		)
	)
)
```


