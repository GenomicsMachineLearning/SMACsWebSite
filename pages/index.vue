<template>
  <div class="min-h-screen bg-gray-100 font-sans flex flex-col items-center">
    <!-- Header -->
    <header class="bg-white shadow w-full">
         <div class="max-w-7xl mx-auto px-6 py-4">
             <h1 class="text-3xl font-bold tracking-tight">
                 <span class="text-[#51158C]">SMACs</span>: <span class="text-gray-800">Spatial Multi-organ Aging Cell-Cell Interactions</span>
             </h1>
         </div>
    </header>

    <div class="flex flex-col md:flex-row max-w-7xl w-full mt-6 px-6 gap-6 flex-grow">
      
      <!-- LEFT SIDEBAR: Controls (Only for Visualiser/Search) -->
      <aside v-if="activeTab !== 'home'" class="w-full md:w-80 bg-white p-6 rounded-xl shadow-lg border border-gray-100 flex-shrink-0 self-start">
        
        <!-- VISUALISER CONTROLS -->
        <div v-if="activeTab === 'visualiser'">
            <h2 class="text-lg font-bold text-gray-800 mb-2 border-b pb-2">DEGs and DELRs on Tissue</h2>
            <p class="text-xs text-gray-600 mb-4 bg-blue-50 p-2 rounded border border-blue-100">
                Choose a Technology and Mode, select a Gene/Interaction, and click on an Organ from the mouse model below to visualize the spatial expression comparing Young and Aged.
            </p>
            
            <!-- Technology -->
            <div class="mb-6">
                <label class="text-sm font-semibold text-gray-600 block mb-2 uppercase tracking-wide">Technology</label>
                <div class="space-y-2">
                    <label v-for="tech in technologies" :key="tech" class="flex items-center p-2 rounded hover:bg-gray-50 cursor-pointer transition-colors">
                        <input type="radio" :value="tech" v-model="selectedTech" @change="refreshData" class="text-blue-600 focus:ring-blue-500 h-4 w-4 border-gray-300">
                        <span class="ml-3 text-gray-700 font-medium">{{ tech }}</span>
                    </label>
                </div>
            </div>

            <!-- Mode -->
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

            <!-- Feature Selection -->
            <div class="mb-6">
                 <label class="text-sm font-semibold text-gray-600 block mb-2 uppercase tracking-wide">
                     {{ visualiserMode === 'Genes' ? 'Select Gene' : 'Select Interaction' }}
                 </label>
                 <select v-model="selectedFeature" @change="fetchPlotsOnly" class="w-full p-2 border border-gray-300 rounded shadow-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
                     <option v-for="feat in availableFeatures" :key="feat" :value="feat">
                         {{ feat }}
                     </option>
                 </select>
            </div>

            <!-- Organ Selection -->
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
                         
                        <!-- Organs -->
                        <g @mouseenter="hoveredOrganName='Brain'" @mouseleave="hoveredOrganName=''" @click="handleOrganClick('Brain')" class="cursor-pointer group/organ">
                             <circle cx="280" cy="200" r="25" :style="{ fill: selectedOrgan === 'Brain' ? '#3b82f6' : '#93c5fd' }" class="organ-dot stroke-[#1d4ed8] stroke-2" />
                        </g>

                        <g @mouseenter="hoveredOrganName='Heart'" @mouseleave="hoveredOrganName=''" @click="handleOrganClick('Heart')" class="cursor-pointer group/organ">
                             <circle cx="300" cy="300" r="25" :style="{ fill: selectedOrgan === 'Heart' ? '#22c55e' : '#86efac' }" class="organ-dot stroke-[#15803d] stroke-2" />
                        </g>

                        <g @mouseenter="hoveredOrganName='Liver'" @mouseleave="hoveredOrganName=''" @click="handleOrganClick('Liver')" class="cursor-pointer group/organ">
                             <circle cx="280" cy="370" r="25" :style="{ fill: selectedOrgan === 'Liver' ? '#f97316' : '#fdba74' }" class="organ-dot stroke-[#c2410c] stroke-2" />
                        </g>

                        <g @mouseenter="hoveredOrganName='Spleen'" @mouseleave="hoveredOrganName=''" @click="handleOrganClick('Spleen')" class="cursor-pointer group/organ">
                             <circle cx="320" cy="430" r="25" :style="{ fill: selectedOrgan === 'Spleen' ? '#a855f7' : '#d8b4fe' }" class="organ-dot stroke-[#7e22ce] stroke-2" />
                        </g>

                        <g @mouseenter="hoveredOrganName='Kidney'" @mouseleave="hoveredOrganName=''" @click="handleOrganClick('Kidney')" class="cursor-pointer group/organ">
                             <circle cx="240" cy="420" r="25" :style="{ fill: selectedOrgan === 'Kidney' ? '#ef4444' : '#fca5a5' }" class="organ-dot stroke-[#b91c1c] stroke-2" />
                        </g>
                    </svg>
                </div>
            </div>
        </div>

        <!-- FIXED STATS BOX at Bottom -->
        <div v-if="activeTab === 'visualiser'" class="mt-auto pt-6 border-t border-gray-100 w-full animate-fade-in">
             <div v-if="loadingStats" class="text-center text-blue-600 animate-pulse text-sm">Loading Stats...</div>
             <div v-else-if="statsData">
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
        </div>

        <!-- SEARCH CONTROLS -->
        <div v-if="activeTab === 'search_genes' || activeTab === 'search_lrs'">
            <h2 class="text-lg font-bold text-gray-800 mb-4 border-b pb-2">Instructions for Search</h2>
            <div class="bg-blue-50 p-4 rounded-lg text-sm text-blue-800 mb-4">
                Select a category in the main window to search through our database of specific aging markers and interactions.
            </div>
        </div>

      </aside>

      <!-- RIGHT CONTENT AREA -->
      <main class="w-full bg-white rounded-xl shadow-lg border border-gray-100 flex flex-col overflow-hidden min-h-[600px] relative">
         
         <!-- TABS HEADER -->
         <div class="flex border-b bg-gray-50">
             <button 
                @click="activeTab = 'home'"
                :class="['flex-1 py-4 text-center font-semibold text-sm focus:outline-none transition-all', activeTab === 'home' ? 'bg-white text-blue-600 border-t-2 border-blue-600' : 'text-gray-500 hover:text-gray-700']"
             >
                HOME
             </button>
             <button 
                @click="activeTab = 'visualiser'"
                :class="['flex-1 py-4 text-center font-semibold text-sm focus:outline-none transition-all', activeTab === 'visualiser' ? 'bg-white text-blue-600 border-t-2 border-blue-600' : 'text-gray-500 hover:text-gray-700']"
             >
                DEGs & DELRs ON TISSUE
             </button>
             <button 
                @click="activeTab = 'search_genes'; selectedCategory = null; searchDataHeaders = []; searchDataResults = []"
                :class="['flex-1 py-4 text-center font-semibold text-sm focus:outline-none transition-all', activeTab === 'search_genes' ? 'bg-white text-blue-600 border-t-2 border-blue-600' : 'text-gray-500 hover:text-gray-700']"
             >
                DIFFERENTIAL GENES
             </button>
             <button 
                @click="activeTab = 'search_lrs'; selectedCategory = null; searchDataHeaders = []; searchDataResults = []"
                :class="['flex-1 py-4 text-center font-semibold text-sm focus:outline-none transition-all', activeTab === 'search_lrs' ? 'bg-white text-blue-600 border-t-2 border-blue-600' : 'text-gray-500 hover:text-gray-700']"
             >
                DIFFERENTIAL LRS
             </button>
         </div>

         <!-- TAB CONTENT: HOME -->
         <div v-if="activeTab === 'home'" class="flex-grow flex flex-col items-center justify-between p-8 bg-white overflow-y-auto">
             <div class="w-full max-w-6xl flex flex-col items-center gap-12 mb-8">
                 <div class="flex flex-col md:flex-row w-full items-center gap-8">
                     <div class="w-full md:w-1/4 flex justify-center">
                         <img src="/SMACs_WebApp_1.png" alt="SMACs Methodology Portrait" class="max-w-full h-auto rounded-lg object-contain" />
                     </div>
                     <div class="w-full md:w-2/3 text-gray-700 text-lg leading-relaxed text-justify">
                         Aging drives systemic functional decline through cell-type- and organ-specific mechanisms. Cell-cell interaction networks within one organ and across organs change with aging, but have not been systematically studied. Here, we leverage information of spatial proximity, ligand-receptor (LR) coexpression, and cell-type co-localisation to accurately map the interactions across five organs and identify changes  between young and aged groups. We generated data from two whole-transcriptome spatial technologies at different spatial resolutions to compare CCI and identify aging-associated interaction pathways across organs. Data from five independent external mouse datasets were used to validate the results. We found increased interactions associated with immune activation, inflammation, and cytokine signalling in aged mice, whereas young mice were enriched for interactions in Wnt signalling differentiation, stemness, regeneration, and tissue homeostasis. Notably, aging-associated LR interactions shared across organs were linked to immune cell recruitment. Spatial mapping of these interactions further identified regions with increased immune interactions, predominantly between immune and organ-resident cells, showing that CCI events collectively orchestrated the inflammatory landscape of the aging organs. Furthermore, using a knowledge graph, we identified associations between multi-organ aging LR genes,  age-related diseases, and corresponding drug targets.
                     </div>
                 </div>
                 
                 <div class="w-full flex justify-center flex-col shadow-lg border border-gray-100 items-center">
                     <h2 class="text-2xl font-bold text-gray-800 mb-4 text-center">Multi-organ conserved aging LR pairs</h2>
                     <img src="/SMACs_WebApp_2.png" alt="SMACs Methodology Landscape" class="max-w-full h-auto drop-shadow-lg rounded-lg" />
                 </div>

                 <div class="w-full mt-12 bg-white p-8 rounded-xl shadow-lg border border-gray-100">
                     <h2 class="text-2xl font-bold text-gray-800 mb-6 text-center">In-house Multi-organ data</h2>
                     <p class="text-center text-gray-600 mb-8">Select an organ to explore its data quality and replicate statistics.</p>
                     
                     <div class="flex flex-wrap justify-center gap-4 mb-10">
                         <button v-for="org in ['Brain', 'Heart', 'Kidney', 'Liver', 'Spleen']" 
                            :key="org" 
                            @click="fetchExploreStats(org)"
                            :class="['px-6 py-2 rounded-full font-semibold transition-all', selectedExploreOrgan === org ? 'bg-blue-600 text-white shadow-md' : 'bg-gray-100 text-gray-700 hover:bg-blue-100']"
                         >
                             {{ org }}
                         </button>
                     </div>
                     
                     <div v-if="loadingExplore" class="text-center text-blue-500 my-10 animate-pulse">
                         Loading organ metadata...
                     </div>
                     
                     <div v-else-if="exploreSummary" class="fade-in">
                         <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-8 bg-gray-50 p-6 rounded-lg">
                             <div class="text-center">
                                 <div class="text-sm text-gray-500 uppercase tracking-widest mb-1">Total Samples</div>
                                 <div class="text-3xl font-bold text-blue-600">{{ exploreSummary.total_samples }}</div>
                             </div>
                             <div class="text-center">
                                 <div class="text-sm text-gray-500 uppercase tracking-widest mb-1">Aged Yes</div>
                                 <div class="text-3xl font-bold text-red-500">{{ exploreSummary.aged_samples }}</div>
                             </div>
                             <div class="text-center">
                                 <div class="text-sm text-gray-500 uppercase tracking-widest mb-1">Aged No</div>
                                 <div class="text-3xl font-bold text-blue-500">{{ exploreSummary.young_samples }}</div>
                             </div>
                             <div class="text-center">
                                 <div class="text-sm text-gray-500 uppercase tracking-widest mb-1">Total Cells</div>
                                 <div class="text-3xl font-bold text-purple-600">{{ exploreSummary.total_cells }}</div>
                             </div>
                         </div>
                         
                         <h3 class="text-lg font-bold text-gray-700 mb-4 border-b pb-2">Average Data Quality (nFeature_Spatial) per Sample</h3>
                         <div class="space-y-3 max-h-96 overflow-y-auto pr-4">
                             <div v-for="stat in exploreStats" :key="stat.orig_ident_2" class="flex items-center gap-4 text-sm">
                                 <div class="w-48 truncate font-medium text-gray-700" :title="stat.orig_ident_2">
                                     {{ stat.orig_ident_2 }}
                                 </div>
                                 <div class="w-16 text-center">
                                     <span :class="['px-2 py-1 text-xs rounded-full font-bold', stat.aged === 'Yes' ? 'bg-red-100 text-red-700' : 'bg-blue-100 text-blue-700']">
                                         {{ stat.aged === 'Yes' ? 'Aged' : 'Young' }}
                                     </span>
                                 </div>
                                 <div class="w-20 text-center">
                                     <span :class="['px-2 py-1 text-xs rounded-md font-semibold border', stat.isVisium ? 'bg-purple-50 text-purple-700 border-purple-200' : 'bg-orange-50 text-orange-700 border-orange-200']">
                                         {{ stat.isVisium ? 'Visium' : 'STOmics' }}
                                     </span>
                                 </div>
                                 <div class="flex-grow bg-gray-100 h-6 rounded-md overflow-hidden relative border border-gray-200">
                                     <!-- Max is around 10000 roughly -->
                                     <div :class="['h-full transition-all', stat.aged === 'Yes' ? 'bg-red-400' : 'bg-blue-400']" :style="{ width: Math.min((stat.nFeature_Spatial / 8000) * 100, 100) + '%' }"></div>
                                     <span class="absolute right-2 top-1 text-xs font-bold text-gray-800 drop-shadow-sm mix-blend-difference">{{ Math.round(stat.nFeature_Spatial) }} features</span>
                                 </div>
                                 <div class="w-24 text-right text-gray-500 text-xs tabular-nums">
                                    {{ stat.cell_count }} cells
                                 </div>
                             </div>
                         </div>
                     </div>
                 </div>
             </div>
             
             <!-- Footer -->
             <div class="w-full flex flex-col items-center mt-10 border-t pt-8">
                 <div class="flex items-center gap-12 mb-6">
                     <img src="/QIMR_logo.png" alt="QIMR Berghofer" class="h-12 object-contain" />
                     <img src="/UQIMB_logo.png" alt="UQ IMB" class="h-12 object-contain" />
                     <img src="/GML_logo.png" alt="GML Lab" class="h-12 object-contain" />
                 </div>
                 <div class="text-center text-sm text-gray-500">
                     <p class="font-bold">Copyright © 2022-2026 GML Lab</p>
                     <p>This project was led by the Genomics and Machine Learning Lab at the QIMR Berghofer Medical Research Institute, Queensland, Australia</p>
                 </div>
             </div>
         </div>

         <!-- TAB CONTENT: VISUALISER -->
         <div v-if="activeTab === 'visualiser'" class="flex-grow p-8 flex flex-col items-center bg-gray-50/50 overflow-y-auto relative">
             
             <div v-if="!selectedOrgan || !selectedTech" class="text-center mt-20 p-8 bg-white border border-gray-200 rounded-lg shadow-sm">
                <h3 class="text-xl font-bold text-gray-700 mb-2">Ready to Visualize</h3>
                <div class="text-gray-700 mb-4">
                    Please use the sidebar controls to select your desired <b>Technology</b> and <b>Mode</b>. 
                    Then click a colored node on the <b>mouse</b> to preview the tissue plots.
                </div>
             </div>
             
             <!-- Loading State: Show Spinner -->
             <div v-else-if="loadingPlots" class="text-center text-blue-600 mt-20 flex flex-col items-center">
                 <svg class="animate-spin h-10 w-10 text-blue-600 mb-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                   <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                   <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                 </svg>
                 <div class="text-xl font-semibold">Loading High-Resolution Plots...</div>
                 <div class="text-sm text-gray-500 mt-2">This may take a few seconds.</div>
             </div>

             <div v-else-if="error" class="bg-red-50 text-red-600 p-4 rounded-lg border border-red-100 mt-10">
                 {{ error }}
             </div>

             <!-- Plots Grid -->
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

         <!-- TAB CONTENT: SEARCH DIFFERENTIAL GENES and LRS -->
         <div v-if="activeTab === 'search_genes' || activeTab === 'search_lrs'" class="flex-grow p-8 flex flex-col">
             
             <!-- Search Categories for Genes (2 Boxes) -->
             <div v-if="activeTab === 'search_genes'" class="grid grid-cols-2 md:grid-cols-2 gap-4 mb-8">
                 <div @click="selectCategory('organ_genes')" :class="['cursor-pointer border rounded-lg p-6 flex flex-col items-center justify-center text-center transition-all hover:shadow-lg', selectedCategory === 'organ_genes' ? 'bg-blue-50 border-blue-500 shadow-md transform scale-105' : 'bg-white hover:border-blue-300']">
                     <span class="text-sm font-bold text-gray-800 mb-2 leading-tight">Organ-specific<br>Genes</span>
                     <img src="/Search_Database_1_3.png" alt="Organ Genes" class="w-25 h-30 object-contain" onerror="this.src='data:image/svg+xml,%3Csvg xmlns=\\\'http://www.w3.org/2000/svg\\\' width=\\\'80\\\' height=\\\'100\\\'%3E%3Crect width=\\\'100%25\\\' height=\\\'100%25\\\' fill=\\\'%23f3f4f6\\\'/%3E%3Ctext x=\\\'50%25\\\' y=\\\'50%25\\\' dominant-baseline=\\\'middle\\\' text-anchor=\\\'middle\\\' font-family=\\\'sans-serif\\\' font-size=\\\'12px\\\' fill=\\\'%239ca3af\\\'%3EImage%3C/text%3E%3C/svg%3E'"/>
                 </div>
                 <div @click="selectCategory('cell_type_degs')" :class="['cursor-pointer border rounded-lg p-6 flex flex-col items-center justify-center text-center transition-all hover:shadow-lg', selectedCategory === 'cell_type_degs' ? 'bg-blue-50 border-blue-500 shadow-md transform scale-105' : 'bg-white hover:border-blue-300']">
                     <span class="text-sm font-bold text-gray-800 mb-2 leading-tight">Cell-type-specific<br>DEGs</span>
                     <img src="/Search_Database_2.png" alt="Cell DEGs" class="w-25 h-30 object-contain" onerror="this.src='data:image/svg+xml,%3Csvg xmlns=\\\'http://www.w3.org/2000/svg\\\' width=\\\'80\\\' height=\\\'100\\\'%3E%3Crect width=\\\'100%25\\\' height=\\\'100%25\\\' fill=\\\'%23f3f4f6\\\'/%3E%3Ctext x=\\\'50%25\\\' y=\\\'50%25\\\' dominant-baseline=\\\'middle\\\' text-anchor=\\\'middle\\\' font-family=\\\'sans-serif\\\' font-size=\\\'12px\\\' fill=\\\'%239ca3af\\\'%3EImage%3C/text%3E%3C/svg%3E'"/>
                 </div>
             </div>

             <!-- Search Categories for LRs (2 Boxes) -->
             <div v-if="activeTab === 'search_lrs'" class="grid grid-cols-2 md:grid-cols-2 gap-4 mb-8">
                 <div @click="selectCategory('de_lr_pairs')" :class="['cursor-pointer border rounded-lg p-6 flex flex-col items-center justify-center text-center transition-all hover:shadow-lg', selectedCategory === 'de_lr_pairs' ? 'bg-blue-50 border-blue-500 shadow-md transform scale-105' : 'bg-white hover:border-blue-300']">
                     <span class="text-sm font-bold text-gray-800 mb-2 leading-tight">Differentially Expressed<br>LR pairs</span>
                     <img src="/Search_Database_1_3.png" alt="DE LR Paris" class="w-25 h-30 object-contain" onerror="this.src='data:image/svg+xml,%3Csvg xmlns=\\\'http://www.w3.org/2000/svg\\\' width=\\\'80\\\' height=\\\'100\\\'%3E%3Crect width=\\\'100%25\\\' height=\\\'100%25\\\' fill=\\\'%23f3f4f6\\\'/%3E%3Ctext x=\\\'50%25\\\' y=\\\'50%25\\\' dominant-baseline=\\\'middle\\\' text-anchor=\\\'middle\\\' font-family=\\\'sans-serif\\\' font-size=\\\'12px\\\' fill=\\\'%239ca3af\\\'%3EImage%3C/text%3E%3C/svg%3E'"/>
                 </div>
                 <div @click="selectCategory('conserved_lr_pairs')" :class="['cursor-pointer border rounded-lg p-6 flex flex-col items-center justify-center text-center transition-all hover:shadow-lg', selectedCategory === 'conserved_lr_pairs' ? 'bg-blue-50 border-blue-500 shadow-md transform scale-105' : 'bg-white hover:border-blue-300']">
                     <span class="text-sm font-bold text-gray-800 mb-2 leading-tight">Conserved Aging<br>Multi-organ LR</span>
                     <img src="/Search_Database_4.png" alt="Conserved LR Pairs" class="w-20 h-24 object-contain" onerror="this.src='data:image/svg+xml,%3Csvg xmlns=\\\'http://www.w3.org/2000/svg\\\' width=\\\'80\\\' height=\\\'100\\\'%3E%3Crect width=\\\'100%25\\\' height=\\\'100%25\\\' fill=\\\'%23f3f4f6\\\'/%3E%3Ctext x=\\\'50%25\\\' y=\\\'50%25\\\' dominant-baseline=\\\'middle\\\' text-anchor=\\\'middle\\\' font-family=\\\'sans-serif\\\' font-size=\\\'12px\\\' fill=\\\'%239ca3af\\\'%3EImage%3C/text%3E%3C/svg%3E'"/>
                 </div>
             </div>

             <!-- Search Interface (Shows only if a category is selected) -->
             <div v-if="selectedCategory" class="flex-grow flex flex-col">
                 <!-- Species Toggle for DE LR Pairs -->
                 <div v-if="selectedCategory === 'de_lr_pairs'" class="mb-4 flex gap-4 items-center bg-gray-50 p-3 rounded-lg border border-gray-100 w-max">
                     <span class="text-sm font-bold text-gray-700 mr-2 text-transform uppercase">Organism:</span>
                     <label class="flex items-center cursor-pointer">
                         <input type="radio" value="mouse" v-model="speciesContext" @change="searchCategoryData" class="text-blue-600 focus:ring-blue-500 h-4 w-4 border-gray-300">
                         <span class="ml-2 text-gray-700 font-medium">Mouse</span>
                     </label>
                     <label class="flex items-center cursor-pointer">
                         <input type="radio" value="human" v-model="speciesContext" @change="searchCategoryData" class="text-blue-600 focus:ring-blue-500 h-4 w-4 border-gray-300">
                         <span class="ml-2 text-gray-700 font-medium">Human</span>
                     </label>
                 </div>

                 <!-- Search Bar -->
                 <div class="mb-6 relative">
                     <input 
                        v-model="searchQuery" 
                        @keyup.enter="searchCategoryData"
                        @input="debounceSearch"
                        placeholder="Search for a gene or feature..." 
                        class="w-full p-4 pl-12 border border-gray-300 rounded-lg shadow-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none text-lg"
                     >
                     <svg class="w-6 h-6 text-gray-400 absolute left-4 top-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path></svg>
                     <button @click="searchCategoryData" class="absolute right-3 top-3 bg-blue-600 text-white px-6 py-2 rounded shadow hover:bg-blue-700 transition">Search</button>
                 </div>

                 <!-- Results Table -->
                 <div class="flex-grow overflow-auto border rounded-lg bg-white shadow-sm" v-if="searchDataHeaders.length > 0 && searchDataResults.length > 0">
                     <table class="min-w-full divide-y divide-gray-200">
                         <thead class="bg-gray-50 sticky top-0">
                             <tr>
                                 <th v-for="header in searchDataHeaders" :key="header" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                    {{ header.replace('.', ' ').replace('_', ' ') }}
                                 </th>
                             </tr>
                         </thead>
                         <tbody class="bg-white divide-y divide-gray-200">
                             <tr v-for="(row, idx) in searchDataResults" :key="idx" class="hover:bg-gray-50">
                                 <td v-for="header in searchDataHeaders" :key="header" class="px-6 py-4 whitespace-nowrap text-sm text-gray-700">
                                     <div v-if="typeof row[header] === 'number'">{{ Number.isInteger(row[header]) ? row[header] : row[header].toFixed(4) }}</div>
                                     <div v-else>{{ row[header] }}</div>
                                 </td>
                             </tr>
                         </tbody>
                     </table>
                 </div>
                 
                 <div v-else-if="isLoadingSearch" class="text-center text-blue-500 mt-10 animate-pulse font-medium">
                     Loading {{ totalMatches !== null && totalMatches > 0 ? 'Results...' : 'Data...' }} 
                 </div>
                 
                 <div v-else-if="hasSearchedData" class="text-center text-gray-500 mt-10">
                     No results found for "{{ lastQuery }}" in this category.
                 </div>
                 
                 <div v-if="totalMatches !== null && !isLoadingSearch && totalMatches > 100" class="text-xs text-center text-gray-500 mt-2 bg-gray-50 p-2 rounded">
                     Showing top 100 out of {{ totalMatches }} records. Please use the search bar to refine.
                 </div>
             </div>
             
             <div v-else class="text-center text-gray-400 mt-10 flex-grow flex items-center justify-center border-2 border-dashed border-gray-200 rounded-lg bg-gray-50">
                 <div>
                    <svg class="w-12 h-12 mx-auto text-gray-300 mb-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 15l-2 5L9 9l11 4-5 2zm0 0l5 5M7.188 2.239l.777 2.897M5.136 7.965l-2.898-.777M13.95 4.05l-2.122 2.122m-5.657 5.656l-2.12 2.122"></path></svg>
                    <span>Click on a category above to load its data table.</span>
                 </div>
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
      activeTab: 'home', // Default to Home
      technologies: ['Visium', 'STOmics'],
      
      // Visualiser State
      selectedOrgan: '',
      selectedTech: 'Visium', 
      visualiserMode: 'Genes', 
      selectedFeature: '', 
      availableFeatures: [],

      plotData: null, 
      statsData: null, 
      loadingPlots: false, // Independent Loader
      loadingStats: false, // Independent Loader
      error: null,
      hoveredOrganName: '',
      cursorX: 0,
      cursorY: 0,

      // Search Tab State
      selectedCategory: null,
      speciesContext: 'mouse',
      searchQuery: '',
      lastQuery: '',
      searchDataResults: [],
      searchDataHeaders: [],
      hasSearchedData: false,
      isLoadingSearch: false,
      totalMatches: null,

      selectedExploreOrgan: null,
      loadingExplore: false,
      exploreStats: null,
      exploreSummary: null,

    }
  },
  mounted() {
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
                if (this.selectedOrgan) {
                    this.refreshData();
                }
            }
        } catch (err) {
            console.error("Error fetching features:", err);
        }
    },
    handleOrganClick(organ) {
        this.selectedOrgan = organ;
        this.activeTab = 'visualiser'; // Switch to visualiser
        this.refreshData();
    },
    refreshData() {
        this.fetchStats();
        this.fetchPlotsOnly();
    },
    async fetchStats() {
        if (!this.selectedOrgan || !this.selectedTech) return;
        this.loadingStats = true;
        this.statsData = null; // Clear old stats immediately
        
        try {
            const response = await this.$axios.get('/stats', {
                params: {
                    organ: this.selectedOrgan,
                    technology: this.selectedTech,
                    mode: this.visualiserMode
                }
            });
            this.statsData = response.data.stats;
        } catch (err) {
            console.error("Error fetching stats:", err);
        } finally {
            this.loadingStats = false;
        }
    },
    async fetchPlotsOnly() {
      if (!this.selectedOrgan || !this.selectedTech || !this.selectedFeature) return;
      
      this.loadingPlots = true;
      this.error = null;
      this.plotData = null; // Clear old plots

      try {
        const response = await this.$axios.get('/plot', {
            params: {
                organ: this.selectedOrgan,
                technology: this.selectedTech,
                feature: this.selectedFeature,
                mode: this.visualiserMode
            }
        });
        
        this.plotData = response.data;
      } catch (err) {
        console.error(err);
        this.error = "Error loading plot";
      } finally {
        this.loadingPlots = false;
      }
    },
    selectCategory(category) {
        this.selectedCategory = category;
        this.searchQuery = '';
        this.searchCategoryData();
    },

    async fetchExploreStats(organ) {
        this.selectedExploreOrgan = organ;
        this.loadingExplore = true;
        this.exploreStats = null;
        this.exploreSummary = null;
        try {
            const response = await this.$axios.get('/explore_organ', {
                params: {
                    organ: organ
                }
            });
            // Format stats a bit
            this.exploreStats = response.data.stats.map(s => ({
                orig_ident_2: s['orig.ident_2'] || s['index'] || 'Unknown',
                aged: s.aged,
                nFeature_Spatial: s.nFeature_Spatial,
                cell_count: s.cell_count,
                tech_org: s.tech_org,
                isVisium: s.tech_org && s.tech_org.includes('VLP')
            })).sort((a,b) => {
                if(a.aged === 'Yes' && b.aged !== 'Yes') return 1;
                if(a.aged !== 'Yes' && b.aged === 'Yes') return -1;
                return b.nFeature_Spatial - a.nFeature_Spatial;
            });
            this.exploreSummary = response.data.summary;
        } catch (err) {
            console.error("Error fetching explore stats:", err);
            this.exploreSummary = { total_samples: 0, aged_samples: 0, young_samples: 0, total_cells: 0 };
            this.exploreStats = [];
        } finally {
            this.loadingExplore = false;
        }
    },
    async searchCategoryData() {
        if (!this.selectedCategory) return;
        this.hasSearchedData = true;
        this.lastQuery = this.searchQuery;
        this.isLoadingSearch = true;
        this.searchDataHeaders = [];
        this.searchDataResults = [];
        
        try {
            const response = await this.$axios.get('/data_search', {
                params: { 
                    category: this.selectedCategory, 
                    query: this.searchQuery,
                    species: this.speciesContext
                }
            });
            this.searchDataResults = response.data.results;
            this.totalMatches = response.data.total_matches;
            
            if (this.searchDataResults.length > 0) {
                // Ignore empty keys or index columns
                let keys = Object.keys(this.searchDataResults[0]);
                this.searchDataHeaders = keys.filter(k => k && k !== 'Unnamed: 0' && k !== 'X');
            }
        } catch(err) {
            console.error("Error searching data:", err);
            this.searchDataResults = [];
            this.totalMatches = 0;
        } finally {
            this.isLoadingSearch = false;
        }
    },
    debounceSearch() {
        if (this.searchTimeout) {
            clearTimeout(this.searchTimeout);
        }
        this.searchTimeout = setTimeout(() => {
            this.searchCategoryData();
        }, 300);
    }
  }
}
</script>
