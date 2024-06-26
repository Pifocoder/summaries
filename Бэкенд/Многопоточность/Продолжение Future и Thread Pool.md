## Future
хотим добавить в нашу [Future](https://gitlab.manytask.org/pcp/students-spring-2024/Pifocoder/-/tree/main/std-future?ref_type=heads) цепочки:
```cpp
std::move(f).Then([](T value){

}).Then([](R value){

}).HandleError([](Error err){

});
```
Потому что такой код проще писать, не нужно постоянно зваться Get.
Есть проблема, каждый отдельный Then - это отдельная Future, не стэк, поэтому прокинуть ошибку до ближайшего HandleError не так просто.
>[!info]
>Then нельзя реализовать  с помошью Get, потому что Then должен быть не локирующим, так как Then возвращает Future, а не её результат.

Теперь в Shared State будем хранить Callback.
Методы, которые понадобятся:
1) AsyncVia - метод, который принимает thread pool и гарантирует, что все настоящие callback будут выполнены в этом thread pool.
2) All - метод, который принимает vector future и возвращает future, которая засабмитится, когда все future в vector засабмитится
3) Any - аналогично All, но только одна из vector.
## Thread Pool
Manual Executor - хорошая замена Thread Pool для тестирования, чтобы что то начало исполняться нужн вызвать Drain.
```cpp
class Manual Executor {
	void Submit(Task t);
	void Drain();//исполняете все такси, которые есть в очереди
private:
	task_queue;
}
```
Inline Executor  - в submit сразу вызывет task.
>[!info]
>На самом деле ThreadPool можно заменить на Executor, чтобы отдебажить, ну или чтобы работать.

Энтрузивный список - будет ссылка на lines TODO
Идея улучшения Thread Pool:
1) У каждого потока сделать отдельную очередь с тасками, это хорошая идея, потому что часто таска порождает новые такски и они уже будут записываться в очередь этого потока
2) когда очередь потока становится пустой, он берет новую таску из общей очереди, а если там нет задач, то бежит по очередям других потоков и пытается им помочь. Может у самого загруженного забрать половину работы.

Fair Share Thread Pool - когда исполняем задачу замеряем CPU время, дальше дается оценка насколько задача тяжелая. Есть отдельные очереди на классы задач(легкие, сложные), делаем кучу на очередях.

Strand - сущность, которая позволяет сабмитить задачи, мы ожидаем, что все исполняться будет на одном потоке. Можно сделать обертку Strand над Thread Pool, но это дз.