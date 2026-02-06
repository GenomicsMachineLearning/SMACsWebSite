<template>
  <div class="min-h-screen bg-gray-100 font-sans flex flex-col items-center">
    <!-- Header -->
    <header class="bg-white shadow w-full">
         <div class="max-w-7xl mx-auto px-6 py-4">
             <h1 class="text-3xl font-bold text-gray-800 tracking-tight">SMACs - Cell-Cell Interactions in Aging</h1>
         </div>
    </header>

    <div class="flex flex-col md:flex-row max-w-7xl w-full mt-6 px-6 gap-6 flex-grow">
      
      <!-- LEFT SIDEBAR: Controls -->
      <aside class="w-full md:w-80 bg-white p-6 rounded-xl shadow-lg border border-gray-100 flex-shrink-0 self-start">
        
        <!-- VISUALISER CONTROLS -->
        <div v-if="activeTab === 'visualiser'">
            <h2 class="text-lg font-bold text-gray-800 mb-4 border-b pb-2">Visualiser Settings</h2>
            
            <!-- Step 1: Technology -->
            <div class="mb-6">
                <label class="text-sm font-semibold text-gray-600 block mb-2 uppercase tracking-wide">Technology</label>
                <div class="space-y-2">
                    <label v-for="tech in technologies" :key="tech" class="flex items-center p-2 rounded hover:bg-gray-50 cursor-pointer transition-colors">
                        <input type="radio" :value="tech" v-model="selectedTech" @change="fetchPlot" class="text-blue-600 focus:ring-blue-500 h-4 w-4 border-gray-300">
                        <span class="ml-3 text-gray-700 font-medium">{{ tech }}</span>
                    </label>
                </div>
            </div>

            <!-- Step 2: Mode (Genes vs LR) -->
            <div class="mb-6">
                <label class="text-sm font-semibold text-gray-600 block mb-2 uppercase tracking-wide">Mode</label>
                <div class="space-y-2">
                    <label class="flex items-center p-2 rounded hover:bg-gray-50 cursor-pointer transition-colors">
                        <input type="radio" value="Genes" v-model="visualiserMode" @change="fetchFeatures" class="text-blue-600 focus:ring-blue-500 h-4 w-4 border-gray-300">
                        <span class="ml-3 text-gray-700 font-medium">Genes</span>
                    </label>
                    <label class="flex items-center p-2 rounded hover:bg-gray-50 cursor-pointer transition-colors">
                        <input type="radio" value="Ligand-Receptor" v-model="visualiserMode" @change="fetchFeatures" class="text-blue-600 focus:ring-blue-500 h-4 w-4 border-gray-300">
                        <span class="ml-3 text-gray-700 font-medium">Ligand-Receptor</span>
                    </label>
                </div>
            </div>

            <!-- Step 3: Feature Selection (Result of Mode) -->
            <div class="mb-6">
                 <label class="text-sm font-semibold text-gray-600 block mb-2 uppercase tracking-wide">
                     {{ visualiserMode === 'Genes' ? 'Select Gene' : 'Select Interaction' }}
                 </label>
                 <select v-model="selectedFeature" @change="fetchPlot" class="w-full p-2 border border-gray-300 rounded shadow-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
                     <option v-for="feat in availableFeatures" :key="feat" :value="feat">
                         {{ feat }}
                     </option>
                 </select>
            </div>

            <!-- Step 4: Organ (Visual Feedback) -->
            <div class="mb-4">
                 <label class="text-sm font-semibold text-gray-600 block mb-2 uppercase tracking-wide">Select Organ</label>
                 <div class="text-xs text-gray-500 mb-2">Click on the mouse diagram to select.</div>
                 <div v-if="selectedOrgan" class="px-3 py-2 bg-blue-50 text-blue-700 rounded-md font-bold text-center border border-blue-200">
                     {{ selectedOrgan }}
                 </div>
                 <div v-else class="px-3 py-2 bg-gray-50 text-gray-400 rounded-md text-center italic border border-gray-200">
                     None Selected
                 </div>
            </div>

            <!-- MOUSE SVG -->
            <div class="flex flex-col items-center mt-6 relative group" @mousemove="updateCursor">
                
                <!-- Cursor Tooltip -->
                <div v-show="hoveredOrganName" 
                     class="fixed z-50 bg-white/80 text-gray-900 backdrop-blur-sm text-xs font-bold px-2 py-1 rounded shadow-xl pointer-events-none transform -translate-x-1/2 -translate-y-full mb-2"
                     :style="{ left: cursorX + 'px', top: (cursorY - 10) + 'px' }">
                    {{ hoveredOrganName }}
                </div>

                <div class="relative w-full max-w-[320px]">
                    <svg viewBox="0 0 595.28 700.89" class="w-full h-auto drop-shadow-sm">
                        <!-- Mouse Outline -->
                        <path d="M285.99,129.29c7.65,10.3,8.26,11.08,9,13,2.69,6.97-.69,8.74,2,17,1,3.08,2.67,5.72,6,11,4.51,7.15,10,14,10,14,0,0,3.85,5.01,8,7,3.9,1.87,5.37-.23,14,0,6.46.17,10.36.27,13,3,3.49,3.61,4,10.99,4,11,0,0-.84,6.77-4,12-3.22,5.32-8.53,8.53-8.54,8.52,0,0,.18-.18.53-.52.95-.91,2.32-2.09,4.25-3.32-3.02,2.54-6.56,6.05-9.83,10.74-1.59,2.28-2.88,4.48-3.92,6.52,1.23-2.48,1.56-3.07,1.59-3.06.08.04-2.8,4.18-3.08,9.12-.22,3.76,1.15,6.57,3.2,10.65,1.61,3.2,4.29,7.71,8.72,12.45.21.35,2.06,3.39,5.5,3.67,1.54.12,2.78-.36,3.44-.69,1.7-.16,3.73-.49,5.96-1.15,3.06-.9,5.52-2.12,7.34-3.21.75-1.35,1.93-2.89,3.84-3.73.25-.11.44-.17.49-.19,0,0,7.4-2.97,14.51-9.81,9.96-9.57,19.03-24.38,20.17-29.48.13-.56.56-2.83,2.38-4.66.89-.89,1.82-1.4,2.41-1.67.45.14,1.1.4,1.77.89,4.47,3.28,2.96,12.21,2.33,15.95,0,0-2.73,14.94-15.79,29.68,0,0-.27.29-.27.29,0,0-2.73,6.29-6,12-2.15,3.76-3.54,6.18-6,9-3.91,4.48-5.13,3.77-9,8-4.94,5.4-3.86,7.53-8,11-3.71,3.11-5.15,1.88-11,5-4.16,2.22-10.71,5.72-13,12-1.66,4.57.87,5.29,0,16-.42,5.19-1.08,5.85-1,10,.07,3.3.53,5.27,2,12,2.71,12.43,2.02,9.89,3,14,2.2,9.28,3.31,13.93,5,18,2.27,5.46,2.85,4.76,6,12,1.98,4.55,2.12,5.65,5,13,2.58,6.57,3.87,9.86,5,12,4.64,8.8,7.87,9.49,10,16,.93,2.83.95,5.89,1,12,.11,13.12-2,25-2,25,0,0-3.8,12.12-11,23-4.98,7.52-7.48,8.26-8,13-.45,4.09,1.26,4.89,1,12-.16,4.28-1,10-1,10,0,0,5.64,2.17,10,3,4.27.82,6.05.12,11,0,5.25-.12,9.58.3,16.03,1.6,2.22.45,4.23.98,4.91,1.16,0,0,2.26.6,4.04,1.23,0,0,0,0,.01,0,1.26,2.69,3.16,5.99,6.06,9.28,1.3,1.48,2.61,2.74,3.85,3.8,1.15.53,2.3,1.06,3.45,1.6,1.31.39,2.97,1.08,4.63,2.32,4.53,3.39,5.7,8.4,6,10-12.06,4.85-19.29,3.56-23.7,1.36-5.63-2.8-6.72-7.09-14.3-9.36-3.67-1.1-4.19-.33-13-2,0,0-4.75-.5-11-3,0,0,0,0,0,0,0,0-14-5-14-5,0,0-6.12-6.87-10-14-2.91-5.36-8.31-15.27-6.25-25.85.46-2.37,1.21-4.34.25-5.15-2.76-2.33-16,8-16,8l-9,14s-3.19,9.13-10,14c-4.91,3.51-12,5-12,5-.02,0,.57-.36.56-.47-.05-.33-5.32.83-10.56.47-1.31-.09-5.88-.05-11.65-1.15-.9-.17-1.64-.33-2.16-.45-1.21-1.1-2.19-2.03-2.88-2.7-3.69-3.58-5.83-6.13-6.32-6.71-1.54-1.82-2.39-3.03-7-10-5.71-8.63-6.11-10.26-9-12-3.01-1.81-3.7-.73-10-3-7.29-2.63-8.27-4.76-10-4-1.27.56-2.52,2.49-1,27,.34,5.39,1,15,1,15-19.91-1.07-24.37.08-24.37.08-6.36,1.64-5.14,3.19-18.63,8.92-5.49,2.33-9.39,3.96-14.75,5.13-5.71,1.24-16.99,3.7-19.25-.13-2.67-4.54,7.35-17.75,19-22,5.41-1.97,11-2,11-2,6.25-.03,7.5,1.85,11.12,1.56,8.5-.69,17.33-12.32,16.88-22.56-.05-1.19-.21-1.85-6-15-3.44-7.8-4.87-10.94-7-16-5.08-12.04-5.68-14.82-6-17-.2-1.37-.63-4.68,0-13,.87-11.55,2.82-20.21,3-21,6.91-30.18,16.52-72.11,29-93,.97-1.62,4.72-7.75,5-16,.09-2.56.2-7.96-3-13-1.36-2.14-3.92-4.12-9-8-5.52-4.21-6.43-4.35-11-8-3.2-2.55-5.42-4.61-8-7-2.87-2.66-6.15-5.7-10-10-3.92-4.38-6.48-7.91-8-10-16.18-22.19-20.6-24.83-25-36-1.7-4.31-6.2-15.74-3-18,4-2.83,16.21,11.43,39,26,7.27,4.65,9.74,5.64,15,10,7.16,5.93,8.23,8.21,11,8,5.6-.42,9.27-10.33,11-15,2.83-7.65,4-17,4-17,0,0-2.42-7.47-6-14-4.22-7.7-6.59-7.77-7-12-.55-5.73,3.35-10.25,4-11,4.02-4.65,9.53-5.58,12-6,5.79-.98,6.88,1.01,12,0,6.58-1.3,10.83-5.77,12-7,2.59-2.72,4.39-7.48,8-17,2.37-6.25,5-14,5-14,0,0,3.17-7.26,7-14,3.96-6.97,10.67-11.02,18-9Z" 
                             fill="none" stroke="#231f20" stroke-width="5" />
                          
                         <!-- Organs (Reduced size slightly to fit interior) -->
                         <g @mouseenter="hoveredOrganName='Brain'" @mouseleave="hoveredOrganName=''" @click="selectedOrgan='Brain'; fetchPlot()" class="cursor-pointer group/organ">
                              <circle cx="280" cy="200" r="25" :style="{ fill: selectedOrgan === 'Brain' ? '#3b82f6' : '#93c5fd' }" 
                              class="organ-dot stroke-[#1d4ed8] stroke-2" />
                         </g>
 
                         <g @mouseenter="hoveredOrganName='Heart'" @mouseleave="hoveredOrganName=''" @click="selectedOrgan='Heart'; fetchPlot()" class="cursor-pointer group/organ">
                              <circle cx="300" cy="300" r="25" :style="{ fill: selectedOrgan === 'Heart' ? '#22c55e' : '#86efac' }" 
                              class="organ-dot stroke-[#15803d] stroke-2" />
                         </g>
 
                         <g @mouseenter="hoveredOrganName='Liver'" @mouseleave="hoveredOrganName=''" @click="selectedOrgan='Liver'; fetchPlot()" class="cursor-pointer group/organ">
                              <circle cx="280" cy="370" r="25" :style="{ fill: selectedOrgan === 'Liver' ? '#f97316' : '#fdba74' }" 
                              class="organ-dot stroke-[#c2410c] stroke-2" />
                         </g>
 
                         <g @mouseenter="hoveredOrganName='Spleen'" @mouseleave="hoveredOrganName=''" @click="selectedOrgan='Spleen'; fetchPlot()" class="cursor-pointer group/organ">
                              <circle cx="320" cy="430" r="25" :style="{ fill: selectedOrgan === 'Spleen' ? '#a855f7' : '#d8b4fe' }" 
                              class="organ-dot stroke-[#7e22ce] stroke-2" />
                         </g>
 
                         <g @mouseenter="hoveredOrganName='Kidney'" @mouseleave="hoveredOrganName=''" @click="selectedOrgan='Kidney'; fetchPlot()" class="cursor-pointer group/organ">
                              <circle cx="240" cy="420" r="25" :style="{ fill: selectedOrgan === 'Kidney' ? '#ef4444' : '#fca5a5' }" 
                              class="organ-dot stroke-[#b91c1c] stroke-2" />
                         </g>
                     </svg>
                 </div>
            </div>
        </div>

        <!-- NEW FIXED STATS BOX at Bottom -->
        <div v-if="statsData" class="mt-auto pt-6 border-t border-gray-100 w-full animate-fade-in">
             
             <!-- Young Stats -->
             <div class="bg-black text-white p-3 rounded mb-4 border-l-4 border-blue-500 shadow-md">
                 <div class="font-bold text-lg text-blue-300 border-b border-gray-700 pb-1 mb-2">{{ statsData.young.header }}</div>
                 <div class="flex justify-between items-center mb-1">
                     <span class="text-gray-400">Samples</span>
                     <span class="text-xl font-mono font-bold">{{ statsData.young.samples }}</span>
                 </div>
                 <div class="text-gray-400 text-sm mb-1">Cell Types</div>
                 <div class="font-mono text-xs whitespace-pre-wrap leading-relaxed text-gray-300">{{ statsData.young.cell_stats }}</div>
             </div>

             <!-- Aged Stats -->
             <div class="bg-black text-white p-3 rounded border-l-4 border-red-500 shadow-md">
                 <div class="font-bold text-lg text-red-300 border-b border-gray-700 pb-1 mb-2">{{ statsData.aged.header }}</div>
                 <div class="flex justify-between items-center mb-1">
                     <span class="text-gray-400">Samples</span>
                     <span class="text-xl font-mono font-bold">{{ statsData.aged.samples }}</span>
                 </div>
                 <div class="text-gray-400 text-sm mb-1">Cell Types</div>
                 <div class="font-mono text-xs whitespace-pre-wrap leading-relaxed text-gray-300">{{ statsData.aged.cell_stats }}</div>
             </div>
        </div>

        <!-- LR DATABASE CONTROLS -->
        <div v-if="activeTab === 'lr_database'">
            <h2 class="text-lg font-bold text-gray-800 mb-4 border-b pb-2">Database Filter</h2>
            <div class="bg-blue-50 p-4 rounded-lg text-sm text-blue-800 mb-4">
                Explore the Ligand-Receptor interactions database.
            </div>
        </div>

      </aside>


      <!-- RIGHT CONTENT AREA -->
      <main class="w-full bg-white rounded-xl shadow-lg border border-gray-100 flex flex-col overflow-hidden min-h-[600px]">
         
         <!-- TABS HEADER -->
         <div class="flex border-b bg-gray-50">
             <button 
                @click="activeTab = 'visualiser'"
                :class="['flex-1 py-4 text-center font-semibold text-sm focus:outline-none transition-all', activeTab === 'visualiser' ? 'bg-white text-blue-600 border-t-2 border-blue-600' : 'text-gray-500 hover:text-gray-700']"
             >
                VISUALISER
             </button>
             <button 
                @click="activeTab = 'lr_database'"
                :class="['flex-1 py-4 text-center font-semibold text-sm focus:outline-none transition-all', activeTab === 'lr_database' ? 'bg-white text-blue-600 border-t-2 border-blue-600' : 'text-gray-500 hover:text-gray-700']"
             >
                LR DATABASE
             </button>
         </div>

         <!-- TAB CONTENT: VISUALISER -->
         <div v-if="activeTab === 'visualiser'" class="flex-grow p-8 flex flex-col items-center bg-gray-50/50 overflow-y-auto">
             
             <div v-if="!selectedOrgan || !selectedTech" class="text-center text-gray-400 mt-20">
                <div class="mb-4">Select an <b>Organ</b> and <b>Technology</b> from the sidebar to start.</div>
             </div>
             
             <div v-else-if="loading" class="text-center text-blue-600 animate-pulse mt-20">
                 Processing Request...
             </div>

             <div v-else-if="error" class="bg-red-50 text-red-600 p-4 rounded-lg border border-red-100 mt-10">
                 {{ error }}
             </div>

             <!-- Multi-Sample Grid -->
             <div v-else-if="plotData" class="w-full max-w-6xl space-y-8">
                 
                 <!-- Row 1: Young Samples -->
                 <section>
                     <h3 class="text-xl font-bold text-gray-800 mb-4 border-b-2 border-blue-200 pb-2 flex items-center">
                        <span class="w-3 h-3 bg-blue-500 rounded-full mr-2"></span> Young Samples
                     </h3>
                     <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                         <div v-for="item in plotData.young" :key="item.id" class="bg-white p-2 rounded shadow border border-gray-100">
                             <img :src="item.image" class="w-full h-auto object-contain rounded" />
                             <div class="mt-2 text-center text-sm font-medium text-gray-600">{{ item.label }}</div>
                         </div>
                     </div>
                 </section>

                 <!-- Row 2: Aged Samples -->
                 <section>
                     <h3 class="text-xl font-bold text-gray-800 mb-4 border-b-2 border-red-200 pb-2 flex items-center">
                        <span class="w-3 h-3 bg-red-500 rounded-full mr-2"></span> Aged Samples
                     </h3>
                     <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                         <div v-for="item in plotData.aged" :key="item.id" class="bg-white p-2 rounded shadow border border-gray-100">
                             <img :src="item.image" class="w-full h-auto object-contain rounded" />
                             <div class="mt-2 text-center text-sm font-medium text-gray-600">{{ item.label }}</div>
                         </div>
                     </div>
                 </section>

             </div>
         </div>

         <!-- TAB CONTENT: LR DATABASE -->
         <div v-if="activeTab === 'lr_database'" class="flex-grow p-8 flex flex-col">
             
             <!-- Search Bar -->
             <div class="mb-6 relative">
                 <input 
                    v-model="searchQuery" 
                    @keyup.enter="searchLR"
                    placeholder="Search for a gene (e.g. Gdf15)..." 
                    class="w-full p-4 pl-12 border border-gray-300 rounded-lg shadow-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none text-lg"
                 >
                 <svg class="w-6 h-6 text-gray-400 absolute left-4 top-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path></svg>
                 <button @click="searchLR" class="absolute right-3 top-3 bg-blue-600 text-white px-6 py-2 rounded shadow hover:bg-blue-700 transition">Search</button>
             </div>

             <!-- Results Table -->
             <div class="flex-grow overflow-auto border rounded-lg bg-white shadow-sm" v-if="searchResults && searchResults.length > 0">
                 <table class="min-w-full divide-y divide-gray-200">
                     <thead class="bg-gray-50">
                         <tr>
                             <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Ligand</th>
                             <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Receptor</th>
                         </tr>
                     </thead>
                     <tbody class="bg-white divide-y divide-gray-200">
                         <tr v-for="(row, idx) in searchResults" :key="idx" class="hover:bg-gray-50">
                             <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">{{ row['Ligand.gene.symbol'] }}</td>
                             <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">{{ row['Receptor.gene.symbol'] }}</td>
                         </tr>
                     </tbody>
                 </table>
             </div>
             
             <div v-else-if="hasSearched" class="text-center text-gray-500 mt-10">
                 No results found for "{{ lastQuery }}".
             </div>

             <div v-else class="text-center text-gray-400 mt-10">
                 Type a gene name above to search the database.
             </div>

         </div>

      </main>

    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      activeTab: 'visualiser', // 'visualiser' or 'lr_database'
      technologies: ['Visium', 'STOmics'],
      
      // Visualiser State
      selectedOrgan: '',
      selectedTech: 'Visium', 
      visualiserMode: 'Genes', 
      selectedFeature: '', 
      availableFeatures: [],

      plotData: null, // Holds { young: [], aged: [], stats: {} }
      statsData: null, // Shortcut to plotData.stats
      loading: false,
      error: null,
      hoveredOrganName: '',
      cursorX: 0,
      cursorY: 0,

      // LR Database State
      searchQuery: '',
      lastQuery: '',
      searchResults: null,
      hasSearched: false
    }
  },
  mounted() {
    // Initial fetch of features
    this.fetchFeatures();
  },
  methods: {
    updateCursor(e) {
        this.cursorX = e.clientX;
        this.cursorY = e.clientY;
    },
    async fetchFeatures() {
        this.availableFeatures = [];
        this.selectedFeature = '';
        try {
            const response = await this.$axios.get('/features', {
                params: { mode: this.visualiserMode }
            });
            this.availableFeatures = response.data;
            if (this.availableFeatures.length > 0) {
                this.selectedFeature = this.availableFeatures[0];
                // Refresh plot if organ is selected
                if (this.selectedOrgan) {
                    this.fetchPlot();
                }
            }
        } catch (err) {
            console.error("Error fetching features:", err);
        }
    },
    async fetchPlot() {
      if (!this.selectedOrgan || !this.selectedTech) return;
      if (!this.selectedFeature) return; // Wait for feature
      
      this.loading = true;
      this.error = null;
      this.plotData = null;
      this.statsData = null;

      try {
        const response = await this.$axios.get('/plot', {
            params: {
                organ: this.selectedOrgan,
                technology: this.selectedTech,
                feature: this.selectedFeature,
                mode: this.visualiserMode // Added mode
            }
        });
        
        // Response is now shape: { young: [...], aged: [...], stats: {...} }
        this.plotData = response.data;
        this.statsData = response.data.stats;

      } catch (err) {
        console.error(err);
        this.error = "Error loading plot";
      } finally {
        this.loading = false;
      }
    },
    async searchLR() {
        if (!this.searchQuery) return;
        this.hasSearched = true;
        this.lastQuery = this.searchQuery;
        
        try {
            const response = await this.$axios.get('/search_lr', {
                params: { query: this.searchQuery }
            });
            this.searchResults = response.data;
        } catch(err) {
            console.error(err);
            this.searchResults = [];
        }
    }
  }
}
</script>

