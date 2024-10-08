# FilamentPHP-Likes-Post-Blog
The source code for the [Filament](https://filamentphp.com) website.

![alt text](https://github.com/Hitsukaya/FilamentPHP-Likes-Post-Blog/blob/main/blog%20post%20like%20button%20filamentphp.png "Like Button")
![alt text](https://github.com/Hitsukaya/FilamentPHP-Likes-Post-Blog/blob/main/Like%20button%20filamentphp.png "Like Button")
![alt text](https://github.com/Hitsukaya/FilamentPHP-Likes-Post-Blog/blob/main/filamentphp-laravel-frontend-who-like-users.png "Like Button")

Developer: Hitsukaya Valentaizar
Email:valentaizarstone@gmail.com / contact@hitsukaya.com 

Install the FireFly Blog 
https://filamentphp.com/plugins/firefly-blog

Add like button for blog post
1. Create migration
```
php artisan make:migration post_like
```

```
<?php

//use App\Models\Post;
use Firefly\FilamentBlog\Models\Post;
use App\Models\User;
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('post_like', function (Blueprint $table) {
            $table->id();
            $table->foreignIdFor(User::class)->index();
            $table->foreignIdFor(Post::class)->index();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('post_like');
    }
};

```
2. Create livewire component
```
php artisan make:livewire LikeButton
```
```
<?php

namespace App\Livewire;

use Livewire\Component;
use Firefly\FilamentBlog\Models\Post;

class LikeButton extends Component
{
    public Post $post;


    public function toggleLike()
    {
        if (auth()->guest())
        {
            return $this->redirect(route('login'), true);
        }

        $user = auth()->user();



        if ($user->hasLiked($this->post))
        {
            $user->likes()->detach($this->post);
            return;
        }

        $user->likes()->attach($this->post);

    }

    public function render()
    {
        return view('livewire.like-button');
    }
}
```
Update
```
 <?php

namespace App\Livewire;

use Livewire\Component;
use Firefly\FilamentBlog\Models\Post;

class LikeButton extends Component
{
    public Post $post;
    //public bool $isLiked;
    public bool $isLiked =false;
    public int $likeCount;
    public $likes;

    protected $listeners = ['likeToggled' => 'refreshLikes'];

    public function refreshLikes()
    {
        $this->post->load('likes');
    }

    // public function mount()
    // {
    //     $this->isLiked = auth()->check() && auth()->user()->hasLiked($this->post);
    // }

    public function mount(Post $post)
    {
        $this->post = $post;
        $this->isLiked = auth()->check() && auth()->user()->hasLiked($this->post);
        $this->likeCount = $this->post->likes()->count();
        $this->likes = $this->post->likes;
    }

    public function toggleLike()
    {
        if (auth()->guest()) {
            return $this->redirect(route('login'), true);
        }

        $user = auth()->user();

        if ($this->isLiked) {
            $user->likes()->detach($this->post);
            $this->isLiked = false;
            $this->likeCount--;
        } else {
            $user->likes()->attach($this->post);
            $this->isLiked = true;
            $this->likeCount++;
        }

        $this->post->load('likes');
        $this->likes = $this->post->likes;
    }


    public function render()
    {
        return view('livewire.like-button',  [
            'isLiked' => $this->isLiked,
            'likeCount' => $this->likeCount,
            'likes' => $this->likes,
        ]);
    }
}


 
```
Update 10 Augsut 2024
```
 <?php

namespace App\Livewire;

use Livewire\Component;
use Firefly\FilamentBlog\Models\Post;

class LikeButton extends Component
{
    public Post $post;
    //public bool $isLiked;
    public bool $isLiked =false;
    public int $likeCount;
    public $likes;
    public bool $isLikesVisible = false;


    protected $listeners = ['likeToggled' => 'refreshLikes'];

    public function refreshLikes()
    {
        $this->post->load('likes');

    }

    public function mount(Post $post)
    {
        $this->post = $post;
        $this->isLiked = auth()->check() && auth()->user()->hasLiked($this->post);
        $this->likeCount = $this->post->likes()->count();
        $this->likes = $this->post->likes;
    }

    public function toggleLike()
    {
        if (auth()->guest()) {

            return redirect()->route('login');
        }

        $user = auth()->user();

        if ($this->isLiked) {
            $user->likes()->detach($this->post);
            $this->isLiked = false;
            $this->likeCount--;
        } else {
            $user->likes()->attach($this->post);
            $this->isLiked = true;
            $this->likeCount++;
        }

        $this->post->load('likes');
        $this->likes = $this->post->likes;
    }

    public function toggleLikesVisibility()
    {
        $this->isLikesVisible = !$this->isLikesVisible;
    }

    public function render()
    {
        return view('livewire.like-button',  [
            'isLiked' => $this->isLiked,
            'likeCount' => $this->likeCount,
            'likes' => $this->likes,
            'isLikesVisible' => $this->isLikesVisible,
        ]);
    }
}
 
```

3. In resources/views/livewire/ ---- Create like-button.blade.php & add this code
```
<button wire.loading.attr="disabled" wire:click="toggleLike()" class="flex items-center">

    <svg wire:loading.delay aria-hidden="true" class="w-8 h-8 text-gray-200 animate-spin fill-red-600"
        viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path
            d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z"
            fill="currentColor" />
        <path
            d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z"
            fill="currentFill" />
    </svg>


    <svg wire:loading.delay.remove xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"
        stroke-width="1.5" stroke="currentColor"
        class="w-6 h-6 {{ Auth::user()?->hasLiked($post) ? 'text-red-600' : 'text-gray-600' }}">
        <path stroke-linecap="round" stroke-linejoin="round"
            d="M21 8.25c0-2.485-2.099-4.5-4.688-4.5-1.935 0-3.597 1.126-4.312 2.733-.715-1.607-2.377-2.733-4.313-2.733C5.1 3.75 3 5.765 3 8.25c0 7.22 9 12 9 12s9-4.78 9-12z" />
    </svg>

    <span class="ml-2 text-gray-600">
        {{ $post->likes()->count() }}
    </span>
</button>
 
```
Update

 ```
<div>
<button wire.loading.attr="disabled" wire:click="toggleLike" class="flex items-center">
    {{-- <span class="bg-red-50 dark:bg-red-50 text-red-500 text-xs font-medium bold me-2 px-2.5 py-2 gap-2 rounded-md dark:text-red-400 border border-red-100">
    @if($isLiked)
      <span class="text-gray-700">Unlike ({{ $likeCount }})</span>
    @else
       <span class="text-red-500">Like ({{ $likeCount }})</span>
    @endif
    </span> --}}



    <svg wire:loading.delay aria-hidden="true" class="w-8 h-8 text-gray-200 animate-spin fill-red-600"
        viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path
            d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z"
            fill="currentColor" />
        <path
            d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z"
            fill="currentFill" />
    </svg>


    <svg wire:loading.delay.remove xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"
        stroke-width="1.5" stroke="currentColor"
        class="w-6 h-6 {{ Auth::user()?->hasLiked($post) ? 'text-red-600' : 'text-gray-600' }}">
        <path stroke-linecap="round" stroke-linejoin="round"
            d="M21 8.25c0-2.485-2.099-4.5-4.688-4.5-1.935 0-3.597 1.126-4.312 2.733-.715-1.607-2.377-2.733-4.313-2.733C5.1 3.75 3 5.765 3 8.25c0 7.22 9 12 9 12s9-4.78 9-12z" />
    </svg>



    <span class="px-2 py-2 ml-2 text-red-500 gap-x-4 dark:text-red-500">
        {{-- {{ $post->likes()->count() }} --}}
        {{ $likeCount }} {{ \Str::plural('like', $likeCount) }}
    </span>




</button>

<div class="py-4 mt-4">
    @foreach ($likes as $like)
        <span class="bg-red-50 dark:bg-red-50 text-red-500 text-xs font-medium bold me-2 px-2.5 py-2 gap-2 rounded-md dark:text-red-400 border border-red-100">
            <svg fill="#000000" height="16px" width="16px" class="inline-flex text-red-500 fill-current" version="1.1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 471.701 471.701">
                <g>
                    <path d="M433.601,67.001c-24.7-24.7-57.4-38.2-92.3-38.2s-67.7,13.6-92.4,38.3l-12.9,12.9l-13.1-13.1c-24.7-24.7-57.6-38.4-92.5-38.4c-34.8,0-67.6,13.6-92.2,38.2c-24.7,24.7-38.3,57.5-38.2,92.4c0,34.9,13.7,67.6,38.4,92.3l187.8,187.8c2.6,2.6,6.1,4,9.5,4c3.4,0,6.9-1.3,9.5-3.9l188.2-187.5c24.7-24.7,38.3-57.5,38.3-92.4C471.801,124.501,458.301,91.701,433.601,67.001z M414.401,232.701l-178.7,178l-178.3-178.3c-19.6-19.6-30.4-45.6-30.4-73.3s10.7-53.7,30.3-73.2c19.5-19.5,45.5-30.3,73.1-30.3c27.7,0,53.8,10.8,73.4,30.4l22.6,22.6c5.3,5.3,13.8,5.3,19.1,0l22.4-22.4c19.6-19.6,45.7-30.4,73.3-30.4c27.6,0,53.6,10.8,73.2,30.3c19.6,19.6,30.3,45.6,30.3,73.3C444.801,187.101,434.001,213.101,414.401,232.701z"/>
                </g>
            </svg>
            {{ $like->name }}
        </span>
    @endforeach
</div>
</div>

```
Update 10 August 2024

 ```
 <div class="px-4 py-4 gap-x-8">

    <div class="flex items-center space-x-4">

        <button wire:click="toggleLike" wire:loading.attr="disabled" class="flex items-center space-x-2">

            <svg wire:loading.delay aria-hidden="true" class="w-8 h-8 text-gray-200 animate-spin fill-red-600"
                viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor" />
                <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill" />
            </svg>


            <svg wire:loading.delay.remove xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"
                class="w-6 h-6 {{ Auth::user()?->hasLiked($post) ? 'text-red-600' : 'text-gray-600' }}">
                <path stroke-linecap="round" stroke-linejoin="round"
                    d="M21 8.25c0-2.485-2.099-4.5-4.688-4.5-1.935 0-3.597 1.126-4.312 2.733-.715-1.607-2.377-2.733-4.313-2.733C5.1 3.75 3 5.765 3 8.25c0 7.22 9 12 9 12s9-4.78 9-12z" />
            </svg>


            <span class="py-2 text-red-500 dark:text-red-500">
                {{ $likeCount }} {{ \Str::plural('like', $likeCount) }}
            </span>
        </button>


        <button wire:click="toggleLikesVisibility" class="px-4 py-2 text-red-500 rounded-md dark:text-red-500">
            {{ $isLikesVisible ? 'Hide User Likes' : 'Show User Likes' }}
        </button>
    </div>


    @if ($isLikesVisible)
        <div class="flex flex-wrap mt-2 space-x-2">
            @foreach ($likes as $like)
                <span class="bg-red-50 dark:bg-red-50 text-red-500 text-xs font-medium bold px-2.5 rounded-md dark:text-red-400 border border-red-100">
                    <svg fill="#000000" height="12px" width="12px" class="inline-flex text-red-500 fill-current" version="1.1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 471.701 471.701">
                        <g>
                            <path d="M433.601,67.001c-24.7-24.7-57.4-38.2-92.3-38.2s-67.7,13.6-92.4,38.3l-12.9,12.9l-13.1-13.1c-24.7-24.7-57.6-38.4-92.5-38.4c-34.8,0-67.6,13.6-92.2,38.2c-24.7,24.7-38.3,57.5-38.2,92.4c0,34.9,13.7,67.6,38.4,92.3l187.8,187.8c2.6,2.6,6.1,4,9.5,4c3.4,0,6.9-1.3,9.5-3.9l188.2-187.5c24.7-24.7,38.3-57.5,38.3-92.4C471.801,124.501,458.301,91.701,433.601,67.001z M414.401,232.701l-178.7,178l-178.3-178.3c-19.6-19.6-30.4-45.6-30.4-73.3s10.7-53.7,30.3-73.2c19.5-19.5,45.5-30.3,73.1-30.3c27.7,0,53.8,10.8,73.4,30.4l22.6,22.6c5.3,5.3,13.8,5.3,19.1,0l22.4-22.4c19.6-19.6,45.7-30.4,73.3-30.4c27.6,0,53.6,10.8,73.2,30.3c19.6,19.6,30.3,45.6,30.3,73.3C444.801,187.101,434.001,213.101,414.401,232.701z" />
                        </g>
                    </svg>
                    {{ $like->name }}
                </span>
            @endforeach
        </div>
    @endif
</div>


```



4. In App\Models\User add this code
```
    public function likes()
    {
        return $this->belongsToMany(Post::class, 'post_like')->withTimestamps();
    }

    public function hasLiked(Post $post)
    {
        return $this->likes()->where('post_id', $post->id)->exists();
    }
```

5. In App\Models\Post or Firefly\FilamentBlog\Models\Post add this code
```
    public function likes()
    {
        return $this->belongsToMany(User::class, 'post_like')->withTimestamps();
    }

    public function scopePopular($query)
    {
        $query->withCount('likes')
        ->orderBy("likes_count", 'desc');
    }

    public function scopeSearch($query, $search = '')
    {
        $query->where('title', 'like', "%{$search}%");
    }
 
```
6. In FireFlyBlog vendor/firefly/src/resources/PostResource or another PostResource add this code
```
                            TextEntry::make('likes.id')
                                ->label('Likes')
                                ->formatStateUsing(fn ($record) => $record->likes->count())
                                ->color('danger')
                                ->icon('heroicon-o-heart')
                                ->iconColor('danger'),

                            TextEntry::make('likes.name') //Show user name of the like post
                                 ->label('Likes Users')
                                 ->color('danger')
                                 ->badge()
                                 ->icon('heroicon-o-heart')
                                 ->iconColor('danger'),
 
```

Update
```
                                            TextEntry::make('likes.id')
                                                ->label('Likes')
                                                ->formatStateUsing(fn ($record) => $record->likes->count())
                                                ->color('danger')
                                                ->icon('heroicon-o-heart')
                                                ->iconColor('danger'),

                                            TextEntry::make('comments.id')
                                                ->label('Comments')
                                                ->color('primary')
                                                ->icon('heroicon-o-chat-bubble-left-right')
                                                ->iconColor('primary')
                                                ->formatStateUsing(fn ($record) => $record->comments->count()),


                                Tabs\Tab::make('Users Like')
                                    ->schema([
                                        TextEntry::make('likes.name') //Show user name of the like post
                                            ->label('Users Like')
                                            ->color('danger')
                                            ->badge()
                                            ->icon('heroicon-o-heart')
                                            ->iconColor('danger'),
                                    ]),
                                ]),
 
```

7. In vendor/filament-blog/blogs/show.blade.php or another blog part add this line
```
Like: <livewire:like-button :key="'like-' . $post->id" :$post />
```
Update
```
<div>
    <span class="inline-flex">Offer Like: <livewire:like-button :post="$post" /><span>
</div>
```

8. In vendor/filament-blog/blogs/show.blade.php or another blog part add this line
```
    <div class="py-2">
    <span>Users like: {{ $post->likes->count() }} <br \><br \>
                        @foreach ($post->likes as $like)
    <span class="bg-red-50 text-red-500 text-xs font-medium bold me-2 px-2.5 py-2 gap-2 rounded dark:bg-gray-700 dark:text-red-400 border border-red-100">
                        <svg fill="#000000" height="17px" width="17px"  class="inline-flex text-red-500 fill-current" version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 471.701 471.701" xml:space="preserve"> <g> <path d="M433.601,67.001c-24.7-24.7-57.4-38.2-92.3-38.2s-67.7,13.6-92.4,38.3l-12.9,12.9l-13.1-13.1  c-24.7-24.7-57.6-38.4-92.5-38.4c-34.8,0-67.6,13.6-92.2,38.2c-24.7,24.7-38.3,57.5-38.2,92.4c0,34.9,13.7,67.6,38.4,92.3   l187.8,187.8c2.6,2.6,6.1,4,9.5,4c3.4,0,6.9-1.3,9.5-3.9l188.2-187.5c24.7-24.7,38.3-57.5,38.3-92.4  C471.801,124.501,458.301,91.701,433.601,67.001z M414.401,232.701l-178.7,178l-178.3-178.3c-19.6-19.6-30.4-45.6-30.4-73.3   s10.7-53.7,30.3-73.2c19.5-19.5,45.5-30.3,73.1-30.3c27.7,0,53.8,10.8,73.4,30.4l22.6,22.6c5.3,5.3,13.8,5.3,19.1,0l22.4-22.4   c19.6-19.6,45.7-30.4,73.3-30.4c27.6,0,53.6,10.8,73.2,30.3c19.6,19.6,30.3,45.6,30.3,73.3 C444.801,187.101,434.001,213.101,414.401,232.701z"/> </g> </svg>
                            {{ $like->name }}
                        </span>
                        @endforeach
                         </span>
                        </div>
 
```
