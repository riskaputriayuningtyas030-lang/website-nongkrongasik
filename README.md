<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tempat Nongkrong Asik - Navigasi Maps</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        // Konfigurasi Tailwind untuk mengaktifkan Dark Mode berdasarkan class 'dark'
        tailwind.config = {
            darkMode: 'class', 
            theme: {
                extend: {
                    colors: {
                        'primary': '#4f46e5', // Indigo 600 - Warna utama
                        'secondary': '#6366f1', // Indigo 500 - Warna sekunder
                    },
                    fontFamily: {
                        // Memastikan font Inter digunakan
                        sans: ['Inter', 'sans-serif'], 
                    },
                }
            }
        }
    </script>
    <style>
        /* Gaya dasar untuk transisi mode gelap */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f7fafc; /* light gray default */
            transition: background-color 0.3s ease, color 0.3s ease;
        }
        /* Background untuk dark mode */
        .dark body {
            background-color: #111827; /* dark gray background */
        }
        
        .star-icon {
            color: #fbbf24; /* yellow-500 */
        }
        
        /* === CARD AESTHETICS & ANIMATION === */
        .place-card {
            box-shadow: 0 10px 20px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05); 
            transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
            cursor: pointer; /* Indikasi bahwa kartu bisa diklik */
        }
        .place-card:hover {
            box-shadow: 0 15px 30px -5px rgba(0, 0, 0, 0.2), 0 5px 10px -3px rgba(0, 0, 0, 0.08);
            transform: translateY(-4px) scale(1.005); /* Efek melayang lebih ringan */
            border-color: #6366f1 !important; /* Warna border saat hover */
        }
        .dark .place-card {
            box-shadow: 0 10px 20px -3px rgba(255, 255, 255, 0.03), 0 4px 6px -2px rgba(255, 255, 255, 0.01);
            border-color: #374151;
        }
        
        @keyframes fadeIn {
            from {
                opacity: 0;
                transform: translateY(20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
        .animated-card {
            animation: fadeIn 0.5s ease-out forwards;
        }

        /* Custom styles for select/input to look good in dark mode */
        select, input[type="text"] {
            @apply bg-white dark:bg-gray-700 dark:text-gray-100 border-gray-300 dark:border-gray-600;
        }
        /* Gaya untuk jam operasional */
        .operational-info {
            @apply flex items-center text-sm font-medium text-gray-700 dark:text-gray-300;
        }
        .operational-info svg {
            @apply w-4 h-4 mr-1 text-primary dark:text-secondary;
        }
        /* Gaya untuk label filter dengan ikon */
        .filter-label-icon {
            @apply flex items-center text-lg font-semibold text-gray-800 dark:text-gray-200 mb-2;
        }
        
        /* === MODAL STYLES === */
        .modal-overlay {
            background-color: rgba(0, 0, 0, 0.7);
            transition: opacity 0.3s ease-in-out;
        }
        .modal-content {
            transform: scale(0.9);
            opacity: 0;
            transition: transform 0.3s ease-in-out, opacity 0.3s ease-in-out;
        }
        .modal-visible .modal-content {
            transform: scale(1);
            opacity: 1;
        }
        .map-frame {
            height: 300px; /* Lebih besar dari sebelumnya */
            width: 100%;
            border-radius: 0.5rem; 
            border: 1px solid #e5e7eb; 
        }
        .dark .map-frame {
            border: 1px solid #374151; 
        }
    </style>
</head>
<body class="min-h-screen transition-colors duration-300">

    <div id="app" class="max-w-7xl mx-auto py-10 px-4 sm:px-6 lg:px-8">
        
        <!-- Header & Dark Mode Toggle -->
        <header class="text-center mb-12 relative">
            <!-- Dark Mode Toggle Button -->
            <button id="mode-toggle" onclick="toggleDarkMode()" 
                    class="absolute top-0 right-0 p-3 rounded-full bg-gray-200 text-gray-800 dark:bg-gray-700 dark:text-gray-200 shadow-lg hover:ring-4 hover:ring-primary dark:hover:ring-secondary transition duration-300 transform hover:scale-110">
                <!-- Sun Icon -->
                <svg id="sun-icon" class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"></path></svg>
                <!-- Moon Icon -->
                <svg id="moon-icon" class="w-6 h-6 hidden" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z"></path></svg>
            </button>
            <h1 class="text-6xl font-black text-primary dark:text-secondary tracking-tighter mb-2">
                NONGKRONG.ID
            </h1>
            <p class="text-xl text-gray-600 dark:text-gray-400 font-light">
                Bosen <span class="text-secondary dark:text-yellow-400 font-medium">*scroll* sosmed?</span> Coba <span class="text-yellow-400 dark:text-secondary font-medium">*refreshing*</span> di *website* nongkrong kita. Dijamin <span class="text-primary dark:text-yellow-400 font-semibold">beda!</span>
            </p>
        </header>

        <!-- Filter, Search, and Sort Area (Sleek Box) -->
        <div class="mb-8 p-6 bg-white dark:bg-gray-800 shadow-2xl rounded-xl border border-gray-100 dark:border-gray-700 transition-colors duration-300">
            
            <!-- HEADER KONTROL DENGAN IKON -->
            <h2 class="text-2xl font-bold text-gray-800 dark:text-gray-200 mb-4 flex items-center">
                <!-- Icon Pengaturan -->
                <svg class="w-6 h-6 mr-2 text-primary dark:text-secondary" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M10.5 4h3a6 6 0 0 0 7 7v3a6 6 0 0 0-7 7h-3a6 6 0 0 0-7-7v-3a6 6 0 0 0 7-7z"></path>
                    <path d="M5 12h2"></path>
                    <path d="M12 5v2"></path>
                    <path d="M17 12h2"></path>
                    <path d="M12 17v2"></path>
                </svg>
                Pencarian & Filter
            </h2>
            
            <!-- SEARCH BAR -->
            <div class="mb-5">
                <label for="search-input" class="filter-label-icon">
                    <!-- Icon Search -->
                    <svg class="w-5 h-5 mr-1 text-gray-500 dark:text-gray-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>
                    Cari Nama/Deskripsi Tempat
                </label>
                <input type="text" id="search-input" onkeyup="updateFiltersAndRender()" placeholder="Contoh: Kopi, Sawah, Rooftop..."
                       class="w-full p-3 border rounded-lg shadow-inner focus:ring-primary focus:border-primary transition duration-150">
            </div>

            <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-5 border-t border-gray-200 dark:border-gray-700 pt-4">
                
                <!-- City Filter (Tujuan) -->
                <div>
                    <label for="city-select" class="filter-label-icon">
                        <!-- Icon Lokasi Tujuan -->
                        <svg class="w-5 h-5 mr-1 text-primary dark:text-secondary" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 10c0 6-9 13-9 13s-9-7-9-13a9 9 0 0 1 18 0z"></path><circle cx="12" cy="10" r="3"></circle></svg>
                        Filter Kota Tujuan
                    </label>
                    <select id="city-select" onchange="updateFiltersAndRender()" class="w-full p-3 border rounded-lg shadow-sm focus:ring-primary focus:border-primary transition duration-150 cursor-pointer">
                        <!-- Opsi kota akan diisi oleh JavaScript -->
                    </select>
                </div>

                <!-- Sort By -->
                <div class="md:col-span-1">
                    <label for="sort-select" class="filter-label-icon">
                        <!-- Icon Urutkan -->
                        <svg class="w-5 h-5 mr-1 text-gray-500 dark:text-gray-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M17 10h-3l3-7zM21 7h-2M15 7h-2"></path><rect x="3" y="10" width="18" height="10" rx="3"></rect><path d="M8 14h-2M16 14h-2M12 14v4"></path></svg>
                        Urutkan Berdasarkan
                    </label>
                    <select id="sort-select" onchange="updateFiltersAndRender()" class="w-full p-3 border rounded-lg shadow-sm focus:ring-primary focus:border-primary transition duration-150 cursor-pointer">
                        <option value="rating-desc">Rating Tertinggi</option>
                        <option value="name-asc">Nama (A-Z)</option>
                    </select>
                </div>
                
                <!-- Placeholder untuk menjaga grid tetap rapi -->
                <div class="hidden md:block"></div> 
                
            </div>
            
            <!-- Category Filter Buttons -->
            <h2 class="filter-label-icon border-t border-gray-200 dark:border-gray-700 pt-4 mt-4">
                <!-- Icon Filter -->
                <svg class="w-5 h-5 mr-1 text-gray-500 dark:text-gray-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polygon points="22 3 2 3 10 12.46 10 19 14 21 14 12.46 22 3"></polygon></svg>
                Filter Kategori
            </h2>
            <div class="flex flex-wrap gap-3">
                <button
                    data-filter="Semua"
                    class="filter-button px-4 py-2 text-sm font-medium rounded-full transition-all duration-300 shadow-lg transform hover:scale-105 active:scale-95 ring-2 ring-transparent hover:ring-primary/50 dark:hover:ring-secondary/50"
                    onclick="filterByCategory('Semua')">
                    Semua
                </button>
                <button
                    data-filter="Kopi"
                    class="filter-button px-4 py-2 text-sm font-medium rounded-full transition-all duration-300 shadow-lg transform hover:scale-105 active:scale-95 ring-2 ring-transparent hover:ring-primary/50 dark:hover:ring-secondary/50"
                    onclick="filterByCategory('Kopi')">
                    Kafe Kopi
                </button>
                <button
                    data-filter="Makanan"
                    class="filter-button px-4 py-2 text-sm font-medium rounded-full transition-all duration-300 shadow-lg transform hover:scale-105 active:scale-95 ring-2 ring-transparent hover:ring-primary/50 dark:hover:ring-secondary/50"
                    onclick="filterByCategory('Makanan')">
                    Restoran/Makanan Berat
                </button>
                <button
                    data-filter="Santai"
                    class="filter-button px-4 py-2 text-sm font-medium rounded-full transition-all duration-300 shadow-lg transform hover:scale-105 active:scale-95 ring-2 ring-transparent hover:ring-primary/50 dark:hover:ring-secondary/50"
                    onclick="filterByCategory('Santai')">
                    Suasana Santai/Alam
                </button>
            </div>
        </div>

        <!-- Recommendation List -->
        <div id="recommendation-list" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8 mt-10">
            <!-- Cards will be injected here by JavaScript -->
        </div>

        <!-- No Results Message (Hidden by default) -->
        <div id="no-results" class="hidden text-center mt-10 p-6 bg-yellow-100 dark:bg-yellow-900 border-l-4 border-yellow-500 text-yellow-700 dark:text-yellow-200 rounded-lg shadow-xl" role="alert">
            <p class="font-bold text-lg">Oops! Pencarian Nihil.</p>
            <p>Tidak ada tempat nongkrong yang cocok dengan filter yang Anda pilih. Coba filter atau kata kunci lain.</p>
        </div>
    </div>
    
    <!-- === MODAL POP-UP DETAIL === -->
    <div id="place-modal" class="modal-overlay hidden fixed inset-0 z-50 flex items-center justify-center p-4" onclick="closeModal()">
        <div class="modal-content w-full max-w-xl bg-white dark:bg-gray-800 rounded-xl shadow-2xl overflow-hidden transform transition-all duration-300" onclick="event.stopPropagation()">
            <!-- Modal Header -->
            <div class="p-4 flex justify-between items-center bg-gray-50 dark:bg-gray-700 border-b border-gray-200 dark:border-gray-600">
                <h2 id="modal-place-name" class="text-3xl font-bold text-gray-900 dark:text-white"></h2>
                <button onclick="closeModal()" class="p-2 text-gray-500 hover:text-red-600 dark:text-gray-300 dark:hover:text-red-400 transition-colors duration-200 rounded-full hover:bg-gray-100 dark:hover:bg-gray-700">
                    <!-- Close Icon (X) -->
                    <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                </button>
            </div>
            
            <!-- Modal Body (Content) -->
            <div class="p-6">
                
                <p class="text-gray-700 dark:text-gray-300 mb-6 text-base" id="modal-place-description"></p>

                <!-- PETA GOOGLE MAPS EMBED -->
                <h3 class="text-xl font-semibold text-gray-900 dark:text-gray-100 mb-3 flex items-center">
                    <svg class="w-5 h-5 mr-2 text-primary dark:text-secondary" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 10c0 6-9 13-9 13s-9-7-9-13a9 9 0 0 1 18 0z"></path><circle cx="12" cy="10" r="3"></circle></svg>
                    Pratinjau Lokasi
                </h3>
                <iframe id="modal-place-map-iframe"
                        title="Pratinjau Lokasi Tempat"
                        class="map-frame mb-6"
                        loading="lazy"
                        allowfullscreen>
                </iframe>
                
                <!-- Detail Info Section -->
                <div class="space-y-3 p-4 bg-gray-50 dark:bg-gray-700 rounded-lg border border-gray-200 dark:border-gray-600">
                    <div id="modal-place-rating"></div>
                    
                    <!-- Jam Buka -->
                    <div id="modal-place-hours" class="flex items-center text-base font-medium text-gray-700 dark:text-gray-300">
                         <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 mr-2 text-primary dark:text-secondary" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
                        </svg>
                        <span>Jam Operasional: <span id="hours-value"></span></span>
                    </div>
                </div>
            </div>

            <!-- Modal Footer -->
            <div class="p-6 bg-gray-50 dark:bg-gray-700 border-t border-gray-200 dark:border-gray-600 flex justify-end">
                <!-- TOMBOL Buka Maps & Lihat Rute -->
                <a id="modal-location-link" href="#" target="_blank" 
                   class="px-5 py-2 bg-green-600 text-white font-bold rounded-lg shadow-lg hover:bg-green-700 transition duration-200 flex items-center transform hover:scale-105 mr-3">
                    Buka Maps & Rute Akurat
                    <svg class="ml-2 w-5 h-5" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C8.13 2 5 5.13 5 9c0 5.25 7 13 7 13s7-7.75 7-13c0-3.87-3.13-7-7-7zM12 11.5c-1.38 0-2.5-1.12-2.5-2.5S10.62 6.5 12 6.5s2.5 1.12 2.5 2.5S13.38 11.5 12 11.5z"/></svg>
                </a>
                <button onclick="closeModal()" 
                        class="px-5 py-2 border border-gray-300 dark:border-gray-600 text-gray-700 dark:text-gray-300 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-700 transition duration-200">
                    Tutup
                </button>
            </div>
        </div>
    </div>
    <!-- === END MODAL POP-UP DETAIL === -->


    <script>
        // State Filter Global
        let currentCityFilter = 'Semua Kota';
        let currentCategory = 'Semua';
        let currentSearch = '';
        let currentSort = 'rating-desc'; 
        
        let renderedPlaces = []; 


        // Data Dummy untuk Tempat Nongkrong (Data Lengkap)
        // VERSI BARU: Dengan nama-nama yang lebih akurat dan koordinat yang disesuaikan
        const placesData = [
            // Madiun (UPDATED)
            { name: "Waroeng Setasiyun Kawak", description: "Tempat vintage yang super unik (Rejomulyo, Kartoharjo). Banyak spot foto, vibes jadul tapi cozy. Selalu ramai karena konsepnya beda dari yang lain.", category: "Santai", rating: 4.7, city: "Madiun", hours: "10:00 - 23:00", image: "https://i.postimg.cc/XN53pmgN/stasiun.webp", coords: [-7.6250, 111.5300] },
            { name: "Ueno Coffee", description: "Estetik, minimalis, vibes modern (Jl. Pahlawan). Banyak dikunjungi mahasiswa & gen Z. Nyaman buat nongkrong maupun nugas (Wi-Fi dan mushola lengkap).", category: "Kopi", rating: 4.8, city: "Madiun", hours: "08:00 - 22:00", image: "https://i.postimg.cc/pT42gtJM/cofe.jpg", coords: [-7.6280, 111.5255] },
            { name: "Hey House Cafe", description: "Tempatnya kekinian, Instagramable (Jl. Sirsat, Kejuron). Indoor–outdoor, cocok buat nongkrong lama. Salah satu yang paling sering muncul di rekomendasi.", category: "Kopi", rating: 4.6, city: "Madiun", hours: "10:00 - 00:00", image: "https://i.postimg.cc/YqtVMsDk/hey.webp", coords: [-7.6360, 111.5220] },
            { name: "Okui! Es Kopi & Temannya", description: "Modern-minimalis, banyak tanaman & aesthetic lighting (Jl. S. Parman). Kopi kekinian, cocok buat anak muda yang suka tempat simpel tapi cakep.", category: "Kopi", rating: 4.5, city: "Madiun", hours: "09:00 - 22:00", image: "https://i.postimg.cc/13kr5gRF/okui.jpg", coords: [-7.6200, 111.5280] },
            { name: "My Story Madiun", description: "Cafe dengan interior stylish (mix modern + klasik) di Jl. Dr. Sutomo. Tempat luas, cocok buat nongkrong ramai-ramai. Sering jadi tujuan hangout.", category: "Santai", rating: 4.6, city: "Madiun", hours: "11:00 - 23:00", image: "https://i.postimg.cc/FsFP2JzM/story.webp", coords: [-7.6230, 111.5210] },
            // --- IMAGE UPDATED ---
            { name: "Swanirwana coffee & grilled chicken", description: "Kafe modern di Madiun yang menawarkan kopi dan ayam panggang sebagai menu utamanya.", category: "Kopi", rating: 4.5, city: "Madiun", hours: "10:00 - 22:00", image: "https://i.postimg.cc/wj7x6V0c/swanira.webp", coords: [-7.6270, 111.5290] },
            { name: "Sthana Coffee", description: "Coffee shop dengan suasana tenang, cocok untuk fokus atau nugas.", category: "Kopi", rating: 4.6, city: "Madiun", hours: "09:00 - 22:00", image: "https://i.postimg.cc/d0598J83/sthana.webp", coords: [-7.6300, 111.5240] },
            { name: "SURPLUS COFFEE STUDIO", description: "Studio kopi dengan konsep minimalis dan modern, menyajikan kopi spesialti.", category: "Kopi", rating: 4.7, city: "Madiun", hours: "08:00 - 21:00", image: "https://i.postimg.cc/R0SWv4rD/surpluds.webp", coords: [-7.6330, 111.5270] },
            // --- IMAGE UPDATED ---
            { name: "SEA Coffee", description: "Tempat nongkrong baru di Madiun dengan kopi dan minuman kekinian.", category: "Kopi", rating: 4.4, city: "Madiun", hours: "10:00 - 23:00", image: "https://i.postimg.cc/Pq1jxFYG/sea.jpg", coords: [-7.6290, 111.5200] },
            
            // Jakarta
            { name: "Kopi Nalar", description: "Kafe di Jakarta Selatan yang nyaman untuk kerja (WFC) dengan kopi berkualitas dan suasana yang tenang.", category: "Kopi", rating: 4.8, city: "Jakarta", hours: "08:00 - 22:00", image: "https://i.postimg.cc/c1tLFtdX/Image-uploaded-from-i-OS.jpg", coords: [-6.2570, 106.8040] }, 
            { name: "Lawless Burgerbar", description: "Restoran burger terkenal dengan konsep *rock and roll* dan porsi yang besar. Sangat populer di kalangan anak muda.", category: "Makanan", rating: 4.5, city: "Jakarta", hours: "11:00 - 23:30", image: "https://i.postimg.cc/LXRGL76S/burger.webp", coords: [-6.2490, 106.8000] },
            { name: "Henshin at The Westin Jakarta", description: "Restoran dan bar rooftop tertinggi di Jakarta. Menawarkan pemandangan 360 derajat kota yang menakjubkan.", category: "Santai", rating: 4.7, city: "Jakarta", hours: "17:00 - 01:00", image: "https://i.postimg.cc/2S0YkCzG/henshin-bar-outdoor.jpg", coords: [-6.2080, 106.8280] },
            { name: "Toko Kopi Tuku - Cipete", description: "Pelopor Kopi Susu Tetangga yang sangat ikonik. Selalu ramai dan menjadi favorit banyak orang.", category: "Kopi", rating: 4.6, city: "Jakarta", hours: "07:00 - 20:00", image: "https://i.postimg.cc/qRzybB7c/tuku.webp", coords: [-6.2610, 106.8070] },
            { name: "Gultik Blok M (Gulai Tikungan)", description: "Kuliner gulai malam hari yang sangat legendaris di area Blok M. Selalu ramai pengunjung.", category: "Makanan", rating: 4.5, city: "Jakarta", hours: "18:00 - 02:00", image: "https://i.postimg.cc/25x0R5rz/gultik.webp", coords: [-6.2440, 106.8020] },
            
            // Yogyakarta
            { name: "Sate Klatak Pak Pong", description: "Sate Klatak paling terkenal di Bantul. Menggunakan jeruji besi, sate ini memiliki rasa khas yang otentik.", category: "Makanan", rating: 4.7, city: "Yogyakarta", hours: "17:00 - 01:00", image: "https://i.postimg.cc/QNWpSBvR/sate-klathak-pak-pong-pusat.webp", coords: [-7.8870, 110.3800] }, 
            { name: "Angkringan Kopi Jos Lik Man", description: "Angkringan legendaris di dekat Stasiun Tugu, pelopor Kopi Jos (kopi dengan arang panas).", category: "Makanan", rating: 4.6, city: "Yogyakarta", hours: "17:00 - 03:00", image: "https://i.postimg.cc/MTHYzdPp/OIP.webp", coords: [-7.7890, 110.3660] },
            { name: "Klinik Kopi", description: "Kafe unik yang terkenal dari film AADC 2. Menawarkan pengalaman *manual brew* yang personal dari ahlinya.", category: "Kopi", rating: 4.8, city: "Yogyakarta", hours: "15:00 - 21:00", image: "https://i.postimg.cc/zBGgnvT8/berinteraksi-dengan-pengunjung.jpg", coords: [-7.7580, 110.3780] },
            { name: "HeHa Sky View", description: "Restoran dan spot foto modern di atas bukit Gukidul. Menawarkan pemandangan kota Jogja yang luas dari ketinggian.", category: "Santai", rating: 4.6, city: "Yogyakarta", hours: "10:00 - 21:00", image: "https://i.postimg.cc/dtsGFWLW/night-vibes.jpg", coords: [-7.8500, 110.4500] },
            { name: "Gudeg Yu Djum Wijilan", description: "Salah satu pusat gudeg paling terkenal dan legendaris di Yogyakarta, berlokasi di sentra gudeg Wijilan.", category: "Makanan", rating: 4.7, city: "Yogyakarta", hours: "08:00 - 21:00", image: "https://i.postimg.cc/L6Y0t6VS/50670443-thumb.jpg", coords: [-7.8040, 110.3680] },
            
            // Bandung
            { name: "Lereng Anteng Panoramic Coffee Place", description: "Kafe di Lembang dengan konsep tenda transparan yang menghadap pemandangan kota Bandung. Sangat instagramable.", category: "Santai", rating: 4.8, city: "Bandung", hours: "09:00 - 21:00", image: "https://i.postimg.cc/3JTtdcYf/teduh-coffee.jpg", coords: [-6.8250, 107.6200] }, 
            { name: "Paskal Food Market", description: "Pusat kuliner outdoor di Paskal 23. Menawarkan ribuan pilihan makanan dari various tenant.", category: "Santai", rating: 4.7, city: "Bandung", hours: "11:00 - 23:00", image: "https://i.postimg.cc/gjpfDkFs/OIP-5.jpg", coords: [-6.9150, 107.5950] },
            { name: "Sejiwa Coffee", description: "Salah satu kafe paling populer di Bandung dengan desain industrial modern dan kopi yang enak.", category: "Kopi", rating: 4.6, city: "Bandung", hours: "07:00 - 22:00", image: "https://i.postimg.cc/5Ng68Xd7/OIP-6.jpg", coords: [-6.8900, 107.6100] },
            { name: "Dusun Bambu", description: "Taman rekreasi keluarga di Lembang dengan berbagai restoran (Lutung Kasarung, Purbasari) di tengah alam yang asri.", category: "Santai", rating: 4.9, city: "Bandung", hours: "10:00 - 18:00", image: "https://i.postimg.cc/gk8xjgNj/OIP-4.jpg", coords: [-6.7860, 107.5600] },
            { name: "Kopi Anjis", description: "Kafe kopi susu yang sangat populer di kalangan mahasiswa dan anak muda Bandung. Harganya terjangkau.", category: "Kopi", rating: 4.5, city: "Bandung", hours: "09:00 - 00:00", image: "https://i.postimg.cc/5NkbBxCv/unnamed-1.jpg", coords: [-6.8830, 107.6150] },
            
            // Surabaya
            { name: "Titik Koma Coffee (Point-Koma)", description: "Kafe populer di Surabaya yang sering dijadikan tempat WFC. Suasananya nyaman dan kopinya enak.", category: "Kopi", rating: 4.5, city: "Surabaya", hours: "07:00 - 21:00", image: "https://i.postimg.cc/FF2YKDCm/Snapinsta-app-339532064-187626727368889-5548277128743007614-n-1080.jpg", coords: [-7.2700, 112.7300] }, 
            { name: "Carpentier Kitchen", description: "Kafe dengan konsep unik yang menyatu dengan *distro*. Punya area outdoor yang nyaman dan *vibey*.", category: "Kopi", rating: 4.4, city: "Surabaya", hours: "09:00 - 23:00", image: "https://i.postimg.cc/pLS4N5Xc/ibnbattuta-xf-P8xak-39Ea2-KWm.jpg", coords: [-7.2800, 112.7400] }, 
            { name: "Sego Sambel Mak Yeye", description: "Street food legendaris Surabaya. Terkenal dengan sambal pedasnya yang nampol dan antrian yang panjang.", category: "Makanan", rating: 4.5, city: "Surabaya", hours: "21:00 - 03:00", image: "https://i.postimg.cc/qqdHPhwx/img-20170927-000433-largejpg.jpg", coords: [-7.2580, 112.7200] }, 
            { name: "Lontong Balap Pak Gendut", description: "Warung Lontong Balap paling terkenal dan legendaris di Surabaya, menyajikan cita rasa otentik.", category: "Makanan", rating: 4.6, city: "Surabaya", hours: "09:00 - 21:00", image: "https://i.postimg.cc/rsbFRFN1/20230821-lontongbalap-aslipakgendut.jpg", coords: [-7.2790, 112.7450] },

            // Semarang
            { name: "Spiegel Bar & Bistro", description: "Kafe dan bar di Gedung Kuno area Kota Lama Semarang. Sangat ikonik dan estetik.", category: "Kopi", rating: 4.7, city: "Semarang", hours: "10:00 - 22:00", image: "https://i.postimg.cc/Bn4BwS0v/koffie-spiegel-heel-goed.jpg", coords: [-6.9690, 110.4280] }, 
            { name: "Toko Kopi Oen", description: "Restoran es krim dan kopi legendaris sejak zaman kolonial. Nuansa klasiknya sangat kental.", category: "Santai", rating: 4.6, city: "Semarang", hours: "10:00 - 21:00", image: "https://i.postimg.cc/P5JPq9bz/20201024-191933-largejpg.jpg", coords: [-6.9820, 110.4200] },
            { name: "Pasar Semawis", description: "Pusat kuliner malam (pecinan) di Semarang yang hanya buka di akhir pekan. Surga jajanan dan makanan.", category: "Makanan", rating: 4.6, city: "Semarang", hours: "18:00 - 23:00 (Jumat-Minggu)", image: "https://i.postimg.cc/4yFR2LYS/Semawis-Semarang.jpg", coords: [-6.9740, 110.4300] },
            
            // Denpasar
            { name: "Revolver Espresso", description: "Salah satu kafe kopi paling terkenal di Bali, tersembunyi di gang kecil Seminyak dengan kualitas kopi *high-end*.", category: "Kopi", rating: 4.8, city: "Denpasar", hours: "07:00 - 23:00", image: "https://i.postimg.cc/tgJrhjFm/16-05-10-16-44-18-403-deco.webp", coords: [-8.6900, 115.1680] }, 
            { name: "Warung Mak Beng Sanur", description: "Restoran legendaris di Sanur yang hanya menyajikan satu menu: Ikan goreng, sup ikan, dan sambal pedas.", category: "Makanan", rating: 4.7, city: "Denpasar", hours: "08:00 - 22:00", image: "https://i.postimg.cc/rsf0Gv1d/unnamed-1.webp", coords: [-8.6950, 115.2600] },
            
            // Medan
            { name: "Ucok Durian", description: "Tempat makan durian paling terkenal di Medan, buka 24 jam. Wajib dikunjungi pecinta durian.", category: "Makanan", rating: 4.7, city: "Medan", hours: "Buka 24 Jam", image: "https://i.postimg.cc/bY5XhnB1/durian-ucok.jpg", coords: [3.5670, 98.6500] }, 
            { name: "Merdeka Walk", description: "Pusat kuliner dan *hangout* terbesar di Medan. Area terbuka dengan banyak pilihan tenant makanan.", category: "Santai", rating: 4.5, city: "Medan", hours: "11:00 - 00:00", image: "https://i.postimg.cc/W1Lgx9mW/1a.jpg", coords: [3.5870, 98.6750] },
            { name: "Soto Medan Sinar Pagi", description: "Warung soto Medan legendaris dengan kuah santan kental dan potongan udang/ayam/daging.", category: "Makanan", rating: 4.7, city: "Medan", hours: "07:00 - 15:00", image: "https://i.postimg.cc/y8Z3P0wb/rm.jpg", coords: [3.5800, 98.6800] },
            { name: "Tip Top Restaurant", description: "Restoran legendaris di Medan dengan nuansa kolonial. Terkenal dengan kue-kue jadul dan *steak*.", category: "Santai", rating: 4.6, city: "Medan", hours: "10:00 - 22:30", image: "https://i.postimg.cc/vBgmYyGF/tip.webp", coords: [3.5900, 98.6740] },
            
            // Makassar
            { name: "Pantai Losari", description: "Ikon kota Makassar. Tempat nongkrong sore hari sambil menikmati sunset dan jajan Pisang Epe.", category: "Santai", rating: 4.8, city: "Makassar", hours: "Buka 24 Jam", image: "https://i.postimg.cc/Y09rjw6j/pantai-losari-makasar.jpg", coords: [-5.1450, 119.4070] }, 
            { name: "Mark Trees Café", description: "Suasana asri dan “go green”, banyak tanaman & dekor kayu. Menu makanan + kopi lengkap, cocok buat nongkrong santai pagi sampai sore.", category: "Santai", rating: 4.6, city: "Makassar", hours: "09:00 - 18:00", image: "https://i.postimg.cc/XYLBtpZR/mark.webp", coords: [-5.1520, 119.4150] },
            { name: "Real Café", description: "Tempat nyaman untuk kerja atau ngobrol lama. Menunya variatif: kopi, makanan berat, hidangan ringan.", category: "Kopi", rating: 4.5, city: "Makassar", hours: "10:00 - 22:00", image: "https://i.postimg.cc/P555pKx6/Real-Cafe.webp", coords: [-5.1480, 119.4250] },
            { name: "Coto Nusantara", description: "Salah satu tempat makan Coto Makassar paling terkenal, disajikan dengan ketupat dan buras.", category: "Makanan", rating: 4.7, city: "Makassar", hours: "08:00 - 22:00", image: "https://i.postimg.cc/x1yT8qKx/20190817-160849-largejpg(1).jpg", coords: [-5.1380, 119.4200] },
            
            // Palembang
            { name: "River Side Restaurant (Benteng Kuto Besak)", description: "Restoran di area Benteng Kuto Besak dengan pemandangan langsung ke Jembatan Ampera dan Sungai Musi.", category: "Santai", rating: 4.7, city: "Palembang", hours: "15:00 - 22:00", image: "https://i.postimg.cc/0Q092yL1/thumbnail.jpg", coords: [-2.9890, 104.7500] }, 
            { name: "Pempek Candy", description: "Salah satu toko pempek paling terkenal di Palembang untuk oleh-oleh atau makan di tempat.", category: "Makanan", rating: 4.6, city: "Palembang", hours: "08:00 - 21:00", image: "https://i.postimg.cc/vm7J5ShJ/download.webp", coords: [-2.9700, 104.7400] },

            // Balikpapan
            { name: "Pantai BSB (Balikpapan Superblock)", description: "Area lifestyle modern di Balikpapan dengan pantai buatan, kafe, dan restoran. Sangat ramai saat akhir pekan.", category: "Santai", rating: 4.6, city: "Balikpapan", hours: "10:00 - 22:00", image: "https://i.postimg.cc/9X1n3642/download.jpg", coords: [-1.2680, 116.8300] },
            { name: "Kopi Kulo - Balikpapan", description: "Cabang Kopi Kulo yang populer. Menyajikan es kopi susu kekinian yang jadi favorit.", category: "Kopi", rating: 4.5, city: "Balikpapan", hours: "10:00 - 23:00", image: "https://i.postimg.cc/XJffwngv/OIP-3.jpg", coords: [-1.2500, 116.8500] },
            { name: "RM Torani (Pusat)", description: "Restoran seafood terbesar dan paling terkenal di Balikpapan, spesialis kepiting dan ikan bakar.", category: "Makanan", rating: 4.7, city: "Balikpapan", hours: "10:00 - 22:00", image: "https://i.postimg.cc/bNR2pS4y/rumah-makan-rm-torani-balikpapan-menu-harga-dan-ulasan.jpg", coords: [-1.2200, 116.8600] },
            
            // Pontianak
            { name: "Tugu Khatulistiwa", description: "Monumen ikonik di Pontianak. Terdapat juga kafe dan area bersantai di sekitar tugu.", category: "Santai", rating: 4.7, city: "Pontianak", hours: "08:00 - 21:00", image: "https://i.postimg.cc/GmkWwhVB/fototugukhatulistiwa.jpg", coords: [0.0010, 109.3200] }, 
            { name: "Warung Kopi Asiang", description: "Warung kopi legendaris di Pontianak yang terkenal dengan kopi saringnya. Selalu ramai di pagi hari.", category: "Makanan", rating: 4.6, city: "Pontianak", hours: "05:00 - 13:00", image: "https://i.postimg.cc/FKKXyXCj/img20180308091516-largejpg.jpg", coords: [-0.0250, 109.3300] },
            { name: "Platform Cafe", description: "Cafe dengan interior unik + banyak spot foto. Ada tema Harry Potter di beberapa bagian, seru buat foto-foto. Menu kopi, makanan ringan, hingga nasi/ayam tersedia.", category: "Kopi", rating: 4.6, city: "Pontianak", hours: "10:00 - 22:00", image: "https://i.postimg.cc/43vLDvv1/platform-cafe.jpg", coords: [-0.0280, 109.3350] },
            { name: "No. 03 Cafe Arts & Lounge", description: "Desain minimalis + taman kecil menjadikan suasananya nyaman dan estetik. Tempat bagus untuk santai, ngobrol, atau sekadar foto.", category: "Santai", rating: 4.5, city: "Pontianak", hours: "11:00 - 23:00", image: "https://i.postimg.cc/C5R4xb8v/cafe-arts.jpg", coords: [-0.0320, 109.3380] },
        ];
        
        // --- DARK MODE LOGIC ---
        function updateModeIcons(isDark) {
            document.getElementById('sun-icon').classList.toggle('hidden', isDark);
            document.getElementById('moon-icon').classList.toggle('hidden', !isDark);
        }

        function toggleDarkMode() {
            const isDark = document.documentElement.classList.toggle('dark');
            localStorage.setItem('theme', isDark ? 'dark' : 'light');
            updateModeIcons(isDark);
        }
        
        // --- LOGIKA UTAMA APLIKASI ---

        // Fungsi untuk menghasilkan bintang rating
        function generateRatingStars(rating) {
            const fullStars = Math.floor(rating);
            const halfStar = rating % 1 >= 0.5;
            const emptyStars = 5 - fullStars - (halfStar ? 1 : 0);
            let starsHtml = '';

            for (let i = 0; i < fullStars; i++) {
                starsHtml += `<span class="star-icon text-yellow-400">?</span>`;
            }

            if (halfStar) {
                starsHtml += `<span class="star-icon text-yellow-400">?</span>`; 
            }

            for (let i = 0; i < emptyStars; i++) {
                starsHtml += `<span class="star-icon text-gray-300 dark:text-gray-600">?</span>`;
            }

            return `<div class="flex items-center space-x-1">${starsHtml} <span class="text-sm font-semibold text-gray-700 dark:text-gray-300 ml-2">${rating.toFixed(1)}</span></div>`;
        }
        
        // Fungsi untuk memberikan warna tag yang konsisten berdasarkan nama kota
        function getCityColor(city) {
            switch (city) {
                case 'Jakarta': return 'bg-blue-600';
                case 'Bandung': return 'bg-red-600';
                case 'Yogyakarta': return 'bg-green-600';
                case 'Surabaya': return 'bg-yellow-600';
                case 'Semarang': return 'bg-purple-600';
                case 'Denpasar': return 'bg-pink-600';
                case 'Madiun': return 'bg-teal-600'; 
                case 'Medan': return 'bg-orange-600';
                case 'Makassar': return 'bg-fuchsia-600';
                case 'Palembang': return 'bg-cyan-600';
                case 'Balikpapan': return 'bg-lime-600';
                case 'Pontianak': return 'bg-indigo-600'; // Warna baru
                default: return 'bg-gray-600';
            }
        }

        // Fungsi untuk membuat card rekomendasi
        function createPlaceCard(place, index) {
            const cityColorClass = getCityColor(place.city);
            
            // Link langsung ke Google Maps dengan mode directions (rute)
            const directionsUrl = `https://www.google.com/maps/dir/?api=1&destination=${place.coords[0]},${place.coords[1]}`;

            // Menggunakan onclick untuk membuka modal
            return `
                <div class="place-card animated-card bg-white dark:bg-gray-900 transition-all duration-300 rounded-xl overflow-hidden border-t-4 border-primary dark:border-secondary" 
                     style="animation-delay: ${index * 0.05}s;"> 
                    <img src="${place.image}" 
                         alt="Gambar ${place.name}" 
                         class="w-full h-48 object-cover"
                         onerror="this.onerror=null; this.src='https://placehold.co/400x250/374151/ffffff?text=Gambar+Tidak+Tersedia';">
                    <div class="p-6">
                        <!-- Tag Kota dan Kategori -->
                        <div class="flex flex-wrap gap-2 mb-3">
                            <span class="px-3 py-1 text-xs font-semibold leading-none rounded-full text-white ${cityColorClass}">${place.city}</span>
                            <span class="px-3 py-1 text-xs font-semibold leading-none rounded-full text-indigo-800 bg-indigo-100 dark:bg-indigo-900 dark:text-indigo-300">${place.category}</span>
                        </div>
                        
                        <h3 class="text-3xl font-extrabold text-gray-900 dark:text-gray-100 mb-3 mt-2">${place.name}</h3>
                        <p class="text-gray-600 dark:text-gray-400 mb-4 text-sm font-light leading-relaxed">${place.description.substring(0, 70)}...</p>
                        
                        <!-- BARIS INFORMASI TAMBAHAN: Rating dan Jam Buka -->
                        <div class="flex flex-col space-y-2 mb-4 border-t border-gray-100 dark:border-gray-700 pt-4">
                            ${generateRatingStars(place.rating)}
                            <div class="operational-info">
                                <!-- Icon Jam (Clock) -->
                                <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
                                </svg>
                                <span>Jam Buka: ${place.hours}</span>
                            </div>
                        </div>

                        <!-- Tombol Langsung Buka Maps -->
                        <a href="${directionsUrl}" target="_blank" class="w-full mt-3 inline-flex justify-center items-center px-4 py-2 bg-green-600 text-white font-bold rounded-lg shadow-md hover:bg-green-700 transition duration-200 transform hover:scale-[1.02]">
                            Buka Maps & Lihat Rute
                            <svg class="ml-2 w-5 h-5" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C8.13 2 5 5.13 5 9c0 5.25 7 13 7 13s7-7.75 7-13c0-3.87-3.13-7-7-7zM12 11.5c-1.38 0-2.5-1.12-2.5-2.5S10.62 6.5 12 6.5s2.5 1.12 2.5 2.5S13.38 11.5 12 11.5z"/></svg>
                        </a>
                        
                        <!-- Tombol Buka Detail Modal -->
                        <button onclick="openModal('${place.name}')" class="w-full mt-2 inline-flex justify-center items-center text-primary dark:text-secondary border border-primary dark:border-secondary hover:bg-primary/10 dark:hover:bg-secondary/20 font-semibold px-4 py-2 rounded-lg transition duration-200">
                            Lihat Detail & Pratinjau Peta
                        </button>

                    </div>
                </div>
            `;
        }
        
        // Fungsi untuk mendapatkan daftar kota unik
        function getUniqueCities(includeAll = false) {
            const cities = new Set(placesData.map(place => place.city));
            const cityList = Array.from(cities).sort();
            return includeAll ? ['Semua Kota', ...cityList] : cityList;
        }

        // Fungsi untuk mengisi dropdown kota
        function renderCityFilters() {
            const filterSelectElement = document.getElementById('city-select');
            
            const cityOptionsWithAll = getUniqueCities(true);

            // Filter Kota Tujuan
            filterSelectElement.innerHTML = ''; 
            cityOptionsWithAll.forEach(city => {
                const option = document.createElement('option');
                option.value = city;
                option.textContent = city;
                filterSelectElement.appendChild(option);
            });
        }

        // Filter berdasarkan kategori (dipanggil dari tombol)
        function filterByCategory(category) {
            currentCategory = category;

            // Update tampilan tombol kategori
            const buttons = document.querySelectorAll('.filter-button');
            buttons.forEach(button => {
                // Hapus kelas aktif dari semua tombol
                button.classList.remove('bg-primary', 'text-white', 'hover:bg-secondary', 'ring-primary', 'ring-offset-2', 'ring-offset-white', 'dark:ring-offset-gray-800');
                // Tambahkan kelas default netral
                button.classList.add('bg-gray-200', 'text-gray-800', 'dark:bg-gray-700', 'dark:text-gray-300', 'hover:bg-gray-300', 'dark:hover:bg-gray-600');

                if (button.getAttribute('data-filter') === category) {
                    // Tombol yang aktif: Warna primary yang tegas + efek ring
                    button.classList.remove('bg-gray-200', 'text-gray-800', 'dark:bg-gray-700', 'dark:text-gray-300', 'hover:bg-gray-300', 'dark:hover:bg-gray-600');
                    button.classList.add('bg-primary', 'text-white', 'hover:bg-secondary', 'ring-primary', 'ring-offset-2', 'ring-offset-white', 'dark:ring-offset-gray-800');
                }
            });

            updateFiltersAndRender();
        }

        
        // Fungsi utama untuk memfilter, mengurutkan, dan merender daftar tempat
        function updateFiltersAndRender() {
            const listContainer = document.getElementById('recommendation-list');
            const noResults = document.getElementById('no-results');
            
            // 1. Ambil state filter terbaru
            currentCityFilter = document.getElementById('city-select').value;
            currentSort = document.getElementById('sort-select').value;
            currentSearch = document.getElementById('search-input').value.toLowerCase();

            // 2. Filter data
            let filteredPlaces = placesData.filter(place => {
                const cityMatch = currentCityFilter === 'Semua Kota' || place.city === currentCityFilter;
                const categoryMatch = currentCategory === 'Semua' || place.category === currentCategory;
                const searchMatch = place.name.toLowerCase().includes(currentSearch) || 
                                    place.description.toLowerCase().includes(currentSearch);
                
                return cityMatch && categoryMatch && searchMatch;
            });
            
            // Simpan hasil filter ke state global untuk akses modal
            renderedPlaces = filteredPlaces;


            // 3. Urutkan data
            if (currentSort === 'rating-desc') {
                renderedPlaces.sort((a, b) => b.rating - a.rating);
            } else if (currentSort === 'name-asc') {
                renderedPlaces.sort((a, b) => a.name.localeCompare(b.name));
            }
            
            // 4. Render tampilan
            listContainer.innerHTML = '';
            noResults.classList.add('hidden');

            if (renderedPlaces.length === 0) {
                noResults.classList.remove('hidden');
                return;
            }
            
            renderedPlaces.forEach((place, index) => {
                listContainer.innerHTML += createPlaceCard(place, index);
            });
            
            listContainer.offsetWidth;
        }

        // --- LOGIKA MODAL POP-UP ---

        function openModal(placeName) {
            // 1. Cari data tempat yang sesuai dari daftar yang sudah dirender
            const place = placesData.find(p => p.name === placeName); // Cari di data lengkap, bukan hanya yang dirender

            if (!place) {
                console.error("Data tempat tidak ditemukan:", placeName);
                return;
            }
            
            const modal = document.getElementById('place-modal');

            // 2. Isi konten modal
            document.getElementById('modal-place-name').textContent = place.name;
            document.getElementById('modal-place-description').textContent = place.description;

            // Rating
            document.getElementById('modal-place-rating').innerHTML = generateRatingStars(place.rating);

            // Jam Buka
            document.getElementById('hours-value').textContent = place.hours;
            
            // Link Maps (untuk tombol "Buka Maps & Lihat Rute" di modal) 
            // Menggunakan mode directions tanpa asal, Maps akan meminta lokasi pengguna
            const directionsUrl = `https://www.google.com/maps/dir/?api=1&destination=${place.coords[0]},${place.coords[1]}`;
            
            document.getElementById('modal-location-link').href = directionsUrl;
            
            // --- SET PETA EMBED (Hanya menampilkan lokasi, bukan rute) ---
            const lat = place.coords[0];
            const lng = place.coords[1];
            // Menggunakan Google Maps Embed API (mode view) dengan penanda
            // NOTE: Perlu API Key asli untuk embed yang berfungsi di luar sandbox
            const mapUrl = `https://www.google.com/maps/embed/v1/view?key=YOUR_API_KEY_HERE&center=${lat},${lng}&zoom=15&maptype=roadmap&q=${lat},${lng}`;
            
            document.getElementById('modal-place-map-iframe').src = mapUrl;

            // 3. Tampilkan modal
            modal.classList.remove('hidden');
            setTimeout(() => {
                modal.classList.add('modal-visible');
            }, 10);
        }

        function closeModal() {
            const modal = document.getElementById('place-modal');
            modal.classList.remove('modal-visible');
            
            setTimeout(() => {
                modal.classList.add('hidden');
            }, 300); 
        }

        // Panggil fungsi saat halaman pertama kali dimuat
        window.onload = () => {
            // Inisialisasi Dark Mode
            const storedTheme = localStorage.getItem('theme');
            const systemPrefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
            let isDark = storedTheme === 'dark' || (!storedTheme && systemPrefersDark);

            if (isDark) {
                document.documentElement.classList.add('dark');
            }
            updateModeIcons(isDark);
            
            // Inisialisasi Filter dan Render
            renderCityFilters();
            // Panggil filterByCategory untuk mengatur tampilan tombol default yang aktif ('Semua')
            filterByCategory(currentCategory); 
            // Panggil render setelah semua state filter di-setup
            updateFiltersAndRender();
        };
    </script>
</body>
</html>
