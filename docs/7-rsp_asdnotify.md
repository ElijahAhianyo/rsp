# Rsp Asdnotify

[_Documentation generated by Documatic_](https://www.documatic.com)

<!---Documatic-section-Codebase Structure-start--->
## Codebase Structure

<!---Documatic-block-system_architecture-start--->
```mermaid
None
```
<!---Documatic-block-system_architecture-end--->

# #
<!---Documatic-section-Codebase Structure-end--->

<!---Documatic-section-rsp.asdnotify.AsyncSystemdNotifier-start--->
## [rsp.asdnotify.AsyncSystemdNotifier](7-rsp_asdnotify.md#rsp.asdnotify.AsyncSystemdNotifier)

<!---Documatic-section-AsyncSystemdNotifier-start--->
<!---Documatic-block-rsp.asdnotify.AsyncSystemdNotifier-start--->
<details>
	<summary><code>rsp.asdnotify.AsyncSystemdNotifier</code> code snippet</summary>

```python
class AsyncSystemdNotifier:

    def __init__(self):
        env_var = os.getenv('NOTIFY_SOCKET')
        self._addr = '\x00' + env_var[1:] if env_var is not None and env_var.startswith('@') else env_var
        self._sock = None
        self._started = False
        self._loop = None
        self._queue = asyncio.Queue(MAX_QLEN)
        self._monitor = False

    @property
    def started(self):
        return self._started

    def _drain(self):
        while not self._queue.empty():
            msg = self._queue.get_nowait()
            self._queue.task_done()
            try:
                self._send(msg)
            except BlockingIOError:
                self._monitor = True
                self._loop.add_writer(self._sock.fileno(), self._drain)
                break
            except OSError:
                pass
        else:
            if self._monitor:
                self._monitor = False
                self._loop.remove_writer(self._sock.fileno())

    def _send(self, data):
        return self._sock.sendto(data, socket.MSG_NOSIGNAL, self._addr)

    async def start(self):
        if self._addr is None:
            return False
        self._loop = asyncio.get_event_loop()
        try:
            self._sock = socket.socket(socket.AF_UNIX, socket.SOCK_DGRAM)
            self._sock.setblocking(0)
            self._started = True
        except OSError:
            return False
        return True

    async def notify(self, status):
        if self._started:
            await self._queue.put(status)
            self._drain()

    async def stop(self):
        if self._started:
            self._started = False
            await self._queue.join()
            if self._monitor:
                self._loop.remove_writer(self._sock.fileno())
            self._sock.close()

    async def __aenter__(self):
        await self.start()
        return self

    async def __aexit__(self, exc_type, exc, traceback):
        await self.stop()
```
</details>
<!---Documatic-block-rsp.asdnotify.AsyncSystemdNotifier-end--->
<!---Documatic-section-AsyncSystemdNotifier-end--->

# #
<!---Documatic-section-rsp.asdnotify.AsyncSystemdNotifier-end--->

[_Documentation generated by Documatic_](https://www.documatic.com)