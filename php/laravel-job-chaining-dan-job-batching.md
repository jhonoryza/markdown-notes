Di Laravel, di antara fitur-fitur yang digunakan untuk menangani proses
asynchronous adalah job chaining dan job batching. Meskipun keduanya digunakan
untuk mengelola pekerjaan dalam antrian (queue), mereka memiliki perbedaan utama
dalam cara mereka menangani dan mengelola pekerjaan tersebut.

## Job Chaining

Job chaining adalah teknik di mana Anda menentukan urutan pekerjaan yang harus
dijalankan secara berurutan. Jika satu pekerjaan gagal, pekerjaan berikutnya
dalam rantai tidak akan dijalankan.

Cara Penggunaan: Kita dapat menggunakan metode chain pada instance Bus untuk
menentukan urutan pekerjaan

```php
use Illuminate\Support\Facades\Bus;
use App\Jobs\JobA;
use App\Jobs\JobB;
use App\Jobs\JobC;

class TestController extends Controller
{
    /**
     * Handle the incoming request.
     */
    public function __invoke(Request $request)
    {
        Bus::chain([
           new JobA,
           new JobB,
           new JobC,
        ])->dispatch();
		}
}
```

Karakteristik:

- Pekerjaan dijalankan secara berurutan.
- Jika salah satu pekerjaan gagal, pekerjaan berikutnya dalam rantai tidak akan
  dijalankan.
- Cocok untuk pekerjaan yang bergantung pada hasil pekerjaan sebelumnya.

## Job Batching

Job batching adalah teknik di mana Anda mengelompokkan beberapa pekerjaan
menjadi satu batch. Pekerjaan dalam batch ini dapat dijalankan secara paralel
atau berurutan tergantung pada konfigurasi. Anda juga dapat menentukan callback
yang akan dijalankan ketika semua pekerjaan dalam batch selesai atau jika ada
pekerjaan yang gagal.

Cara Penggunaan: Kita dapat menggunakan metode batch pada instance Bus untuk
membuat batch pekerjaan.

```php
use Illuminate\Support\Facades\Bus;
use App\Jobs\JobA;
use App\Jobs\JobB;
use App\Jobs\JobC;

class TestController extends Controller
{
    /**
     * Handle the incoming request.
     */
    public function __invoke(Request $request)
    {
       Bus::batch([
         new JobA,
         new JobB,
         new JobC,
       ])->then(function (Batch $batch) {
           // All jobs completed successfully...
       })->catch(function (Batch $batch, Throwable $e) {
          // First batch job failure detected...
       })->finally(function (Batch $batch) {
         // The batch has finished executing...
       })->dispatch();
		}
}
```

Karakteristik:

- Pekerjaan dalam batch dapat dijalankan secara paralel atau berurutan.
- Kita dapat menentukan callback untuk menangani keberhasilan atau kegagalan
  pekerjaan.
- Cocok untuk pekerjaan yang dapat dijalankan secara independen tetapi perlu
  dikelompokkan untuk pemantauan atau pengelolaan.

## Ringkasan Perbedaan

- Job Chaining: Mengatur urutan pekerjaan yang harus dijalankan secara
  berurutan. Jika satu pekerjaan gagal, pekerjaan berikutnya tidak akan
  dijalankan.
- Job Batching: Mengelompokkan beberapa pekerjaan menjadi satu batch yang dapat
  dijalankan secara paralel atau berurutan. Anda dapat menentukan callback untuk
  menangani hasil batch.

Pilihan antara job chaining dan job batching tergantung pada kebutuhan spesifik
aplikasi Anda dan bagaimana Anda ingin mengelola serta menangani pekerjaan dalam
antrian.

## kombinasi bus chain dan bus batch

```php
class TestController extends Controller
{
    /**
     * Handle the incoming request.
     */
    public function __invoke(Request $request)
    {
        Bus::chain([
            function () {
                Bus::batch([
                    new TestJob(1),
                    new TestJob(2),
                    new TestJob(3),
                    new TestJob(4),
                    new TestJob(5),
                    new TestJob(6),
                ])->then(function (Batch $batch) {
                    Log::alert('All Test job completed successfully... batch id '.$batch->id);
                })->dispatch();
            },
            new NotifJob,
            new TestUpdateJob,
            new AnotherUpdateJob,
        ])->dispatch();
			
		}
}
```

atau

```php
class TestController extends Controller
{
    /**
     * Handle the incoming request.
     */
    public function __invoke(Request $request)
    {
        Bus::batch([
            new TestJob(1),
            new TestJob(2),
            new TestJob(3),
            new TestJob(4),
            new TestJob(5),
            new TestJob(6),
        ])->then(function (Batch $batch) {

            Log::alert('All Test Job completed successfully...');

            Bus::chain([
                new NotifJob,
                new TestUpdateJob,
                new AnotherUpdateJob,
            ])->dispatch();
        })->dispatch();
			
		}
}
```

atau implementasi chain di lakukan di dalam class job

```php
class NotifJob implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct()
    {
        //
    }

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        // Set the jobs that should run if this job is successful.
        // this will override
        $this->chain([
					new PosUpdateJob
				]);

        // Prepend a job to the current chain so that it is run after the currently running job.
        $this->prependToChain([
					new PosUpdateJob
				]);

        //Append a job to the end of the current chain.
        $this->appendToChain([
					new PosUpdateJob
				]);
		}
}

class TestController extends Controller
{
    /**
     * Handle the incoming request.
     */
    public function __invoke(Request $request)
    {
        Bus::chain([
            function () {
                Bus::batch([
                    new TestJob(1),
                    new TestJob(2),
                    new TestJob(3),
                    new TestJob(4),
                    new TestJob(5),
                    new TestJob(6),
                ])->then(function (Batch $batch) {
                    Log::alert('All Test Job completed successfully... batch id '.$batch->id);
                })->dispatch();
            },
            new NotifJob,
            new OmsUpdateJob,
        ])->dispatch();
		}
}
```
