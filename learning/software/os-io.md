# How Linux OS handle IO ??

## Select/Poll/Epoll syscall

these 3 syscall all generally poll a file descriptor to check if there's data to be read or not, the difference lies in implementation and performance etc, the concept is the same.

generally, your app will have a main loop that keep polling the file descriptor you're interested and read() and process data if there's any


## Async vs Non-blocking

Async is just the API, you can have async api with blocking mechanism that blocks the underlying thread and still gives your app control that allows concurrent operation, but then your concurrent ability will be limited to thread pool size

Non-blocking io is only useful with async api, else you end up with app code blocking but io is non-blocking.



