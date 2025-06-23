<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>iGaming 경쟁사 소셜 미디어 분석 대시보드</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Cool Neutrals with Blue and Green Accents -->
    <!-- Application Structure Plan: The SPA is a dashboard focused on social media competitive analysis for the iGaming industry. The structure is hierarchical: 1) Top-level KPIs for a quick overview. 2) Visualizations (Bar & Doughnut charts) for comparative analysis of followers and industry segments. 3) A detailed, interactive table showing company-specific data. A key interaction is the expandable row in the table, which reveals the top posts for each company. This structure allows users to seamlessly move from a macro market view to micro-level content analysis without leaving the page. Global filters for 'Industry' and a search bar provide powerful data slicing capabilities. -->
    <!-- Visualization & Content Choices: 
        1. Follower Counts (Report Info: 팔로워 수) -> Goal: Compare -> Viz: Horizontal Bar Chart (Chart.js) -> Interaction: Dynamically updates on filter, tooltips show exact numbers -> Justification: Best for comparing values across many categories (companies).
        2. Industry Distribution (Report Info: 업종) -> Goal: Show composition -> Viz: Doughnut Chart (Chart.js) -> Interaction: Updates on filter, tooltips show percentages -> Justification: Clearly shows the proportion of each industry segment in the dataset.
        3. Company & Post Details (Report Info: Full table) -> Goal: Organize & Explore -> Viz: Interactive HTML Table with expandable rows -> Interaction: Sortable columns, expandable rows to show post details, filterable via global controls -> Justification: The most effective way to present detailed, multi-layered data (company stats + individual posts) in a compact and user-friendly format.
    -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Noto Sans KR', sans-serif;
            background-color: #f4f7f6;
        }
        .chart-container {
            position: relative;
            width: 100%;
        }
        .table-sortable th {
            cursor: pointer;
        }
        .table-sortable th:hover {
            background-color: #e9eef2;
        }
        .table-sortable th .sort-indicator {
            display: inline-block;
            margin-left: 8px;
            opacity: 0.5;
            transition: opacity 0.2s;
        }
        .post-details {
            display: none;
            background-color: #fafafa;
        }
        .post-details.open {
            display: table-row;
        }
        .toggle-btn {
            transition: transform 0.3s ease;
        }
        .open .toggle-btn {
            transform: rotate(90deg);
        }
    </style>
</head>
<body class="text-gray-800">

    <div class="container mx-auto p-4 sm:p-6">
        <header class="text-center mb-8">
            <h1 class="text-2xl sm:text-3xl font-bold text-gray-900">iGaming 소셜 미디어 분석</h1>
            <p class="mt-2 text-sm sm:text-base text-gray-600">경쟁 환경 및 콘텐츠 참여도 분석</p>
        </header>

        <div class="bg-white p-4 rounded-xl shadow-lg mb-6">
            <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div>
                    <label for="industryFilter" class="block text-sm font-medium text-gray-700 mb-1">업종 선택</label>
                    <select id="industryFilter" class="w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500">
                        <option value="All">전체</option>
                    </select>
                </div>
                <div>
                    <label for="searchCompany" class="block text-sm font-medium text-gray-700 mb-1">업체명 검색</label>
                    <input type="text" id="searchCompany" placeholder="업체명 입력..." class="w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500">
                </div>
                <div class="flex items-end">
                    <button id="resetButton" class="w-full bg-gray-600 text-white px-4 py-2 rounded-md hover:bg-gray-700 transition">필터 초기화</button>
                </div>
            </div>
        </div>
        
        <section id="kpi-overview" class="grid grid-cols-2 lg:grid-cols-4 gap-4 mb-8">
            <div class="bg-white p-4 rounded-xl shadow-lg text-center">
                <h3 class="text-sm font-semibold text-gray-500">총 분석 업체</h3>
                <p id="totalCompanies" class="text-2xl sm:text-3xl font-bold text-blue-600 mt-1">0</p>
            </div>
            <div class="bg-white p-4 rounded-xl shadow-lg text-center">
                <h3 class="text-sm font-semibold text-gray-500">총 팔로워</h3>
                <p id="totalFollowers" class="text-2xl sm:text-3xl font-bold text-blue-600 mt-1">0</p>
            </div>
            <div class="bg-white p-4 rounded-xl shadow-lg text-center col-span-2 sm:col-span-1">
                <h3 class="text-sm font-semibold text-gray-500">최고 성장 업체</h3>
                <p id="topGrowthCompany" class="text-lg sm:text-xl font-bold text-green-600 mt-1 truncate">-</p>
            </div>
             <div class="bg-white p-4 rounded-xl shadow-lg text-center col-span-2 sm:col-span-1">
                <h3 class="text-sm font-semibold text-gray-500">최다 참여 게시물</h3>
                <p id="topPostCompany" class="text-lg sm:text-xl font-bold text-green-600 mt-1 leading-tight truncate">-</p>
            </div>
        </section>


        <main class="grid grid-cols-1 lg:grid-cols-5 gap-6 mb-8">
            <div class="lg:col-span-3 bg-white p-4 rounded-xl shadow-lg">
                <h3 class="text-lg font-bold text-center mb-4">업체별 팔로워 수</h3>
                <div class="chart-container h-[350px] md:h-[400px]">
                    <canvas id="followerChart"></canvas>
                </div>
            </div>
            <div class="lg:col-span-2 bg-white p-4 rounded-xl shadow-lg">
                <h3 class="text-lg font-bold text-center mb-4">업종 분포</h3>
                <div class="chart-container h-[300px] md:h-[400px]">
                    <canvas id="industryChart"></canvas>
                </div>
            </div>
        </main>
        
        <section id="competitor-details">
             <h2 class="text-xl font-bold mb-4 border-l-4 border-blue-500 pl-3">업체별 상세 데이터 및 인기 게시물</h2>
              <p class="mb-4 text-gray-600 text-sm">아래 표에서 각 업체의 상세 데이터를 확인하고, 행을 클릭하여 인기 게시물을 펼쳐볼 수 있습니다.</p>
            <div class="bg-white rounded-xl shadow-lg overflow-x-auto">
                <table class="w-full text-sm text-left text-gray-500 table-sortable">
                    <thead class="text-xs text-gray-700 uppercase bg-gray-100">
                        <tr>
                            <th class="p-2 w-10"></th>
                            <th scope="col" class="px-3 py-3 md:px-6" data-sort="name">업체명<span class="sort-indicator"></span></th>
                            <th scope="col" class="px-3 py-3 md:px-6" data-sort="followers">팔로워<span class="sort-indicator"></span></th>
                            <th scope="col" class="px-3 py-3 md:px-6" data-sort="growth">증감<span class="sort-indicator"></span></th>
                        </tr>
                    </thead>
                    <tbody id="companyTableBody">
                    </tbody>
                </table>
            </div>
        </section>
    </div>

    <script>
        const companyData = [
            { id: 'slotsLaunch', name: 'Slots Launch', industry: '미디어/어필리에이트', followers: 25145, growth: 35, posts: [
                { content: 'Here are the new demos that were added to our platform in the last 24-hours: 🥈 Big Snapper - GameArt 🥈 Crab Attack - Betixon 🥈 Freedom Eagle Mega X - Gaming Corps...', hashtags: '#iGaming', likes: 0, comments: 0, shares: 0 },
                { content: 'Here are the new demos that were added to our platform in the last 24-hours: 🥈 3 Witch\'s Lamp - TaDa Gaming | Game Provider 🥈 4 FOXX Crew WildEnergy MultiMax - Yggdrasil...', hashtags: '#iGaming', likes: 0, comments: 0, shares: 0 },
                { content: 'Here are the new demos that were added to our platform in the last 24-hours: 🥈 Alien Invaders - Pragmatic Play 🥈 Bountiful Birds - Microgaming 🥈 Fu Wu Shi Gold Blitz Ultimate...', hashtags: '#iGaming', likes: 0, comments: 0, shares: 0 }
            ]},
            { id: 'bigwinboard', name: 'Bigwinboard®', industry: '미디어/어필리에이트', followers: 24497, growth: 24, posts: [
                { content: '🔥 Top Trending Slots on Bigwinboard Week 24, 2025: 1. Gates of Olympus Super Scatter by Pragmatic Play 2. Pirots 3 by ELK Studios...', hashtags: '#igaming, #gambling, #twitch, #slots, #onlinecasino, #casinostreaming, #youtube', likes: 33, comments: 2, shares: 0 },
                { content: '🔥 Top Trending Slots on Bigwinboard Week 23, 2025: 1. Gates of Olympus Super Scatter by Pragmatic Play 2. Pirots 3 by ELK Studios...', hashtags: '#igaming, #gambling, #twitch, #slots, #onlinecasino, #casinostreaming, #youtube', likes: 42, comments: 1, shares: 0 },
                { content: '🔥 Top Trending Slots on Bigwinboard Week 22, 2025: 1. Gates of Olympus Super Scatter by Pragmatic Play 2. Pirots 3 by ELK Studios...', hashtags: '#igaming, #gambling, #twitch, #slots, #onlinecasino, #casinostreaming, #youtube', likes: 30, comments: 2, shares: 0 }
            ]},
            { id: 'amigoGaming', name: 'Amigo Gaming', industry: 'CP', followers: 109755, growth: 41, posts: [
                { content: '🤠 Curious about what makes Coinboy Riches unique? 🤔 Casino.Online covers it all in this detailed game review...', hashtags: '#Slots, #iGaming, #BigWin, #OnlineCasino, #AmigoGaming, #Ibiza', likes: 7, comments: 0, shares: 0 },
                { content: '🤠 Exciting times ahead! 🔈 Our Chief Amigo, Igor Rus, will take the stage at the upcoming IGAMING BUSINESS ZONE (IBZ) event in Ibiza...', hashtags: '#Slots, #iGaming, #BigWin, #OnlineCasino, #AmigoGaming, #Ibiza', likes: 34, comments: 0, shares: 1 },
                { content: 'Always great to connect with passionate people in the industry. Today I had a pleasure to chat with Mkhitar Kafyan from TotoGaming...', hashtags: '#SBC, #Totogaming, #AmigoGaming, #Partnerships, #GamingIndustry', likes: 38, comments: 1, shares: 2 }
            ]},
            { id: 'spribe', name: 'SPRIBE', industry: 'CP/어그리게이터', followers: 524411, growth: -8, posts: [
                { content: 'David Natroshvili, Founder and CEO of SPRIBE, ranks in the TOP 100 Most Influential People in iGaming! 🚀', hashtags: '#8, #SPRIBE, #Aviator, #iGaming', likes: 103, comments: 4, shares: 1 },
                { content: 'What makes SPRIBE different? Our Turbo games deliver smooth gameplay with instant feedback...', hashtags: '#SPRIBE, #Games, #iGaming', likes: 6, comments: 0, shares: 0 },
                { content: 'New Opportunities at SPRIBE 📣 We\'re hiring across multiple departments. Join us where innovation, collaboration, and creating exceptional gaming experiences are valued.', hashtags: '#SPRIBE, #Aviator, #BroadwayPlatfrom, #iGaming, #OpenRoles', likes: 25, comments: 0, shares: 6 }
            ]},
            { id: 'bgaming', name: 'BGaming', industry: 'CP/어그리게이터', followers: 716296, growth: 171, posts: [
                { content: 'We thought — why not switch things up at iGB L!VE London by inviting you straight into the BGaming Office instead of a noisy booth?...', hashtags: '#SG12,, #igblive2025, #BGaming', likes: 26, comments: 2, shares: 0 },
                { content: 'Kick off the sports summer with BGaming’s new 𝐅𝐨𝐨𝐭𝐛𝐚𝐥𝐥 𝐏𝐥𝐢𝐧 𝐨! ⚽...', hashtags: '', likes: 26, comments: 0, shares: 1 },
                { content: 'Time for a blooming release from BGaming: 𝐌𝐲𝐬𝐭𝐞𝐫𝐲 𝐆𝐚𝐫𝐝𝐞𝐧! 💐...', hashtags: '', likes: 25, comments: 1, shares: 1 }
            ]},
            { id: 'lightWonder', name: 'Light & Wonder Spark', industry: '어그리게이터', followers: 3834, growth: 3, posts: [
                 { content: 'Introducing Bitz - Next-Level Crash Games from our Light & Wonder Spark studio partner, Ricochet BD. 🎮 Dynamic & Engaging...', hashtags: '#ItsAllAboutTheGames', likes: 80, comments: 5, shares: 0 },
                { content: 'Check out the incredible lineup of games set for release across the UK and EU in the coming weeks and months...', hashtags: '#ItsAllAboutTheGames', likes: 60, comments: 3, shares: 0 },
                { content: 'Take a first look at the roadmap of exciting new content set to release across North America in January, February, and beyond...', hashtags: '#ItsAllAboutTheGames', likes: 54, comments: 1, shares: 2 }
            ]},
            { id: 'gamesGlobal', name: 'Games Global', industry: '오퍼레이터', followers: 535864, growth: 235, posts: [
                 { content: 'We’re proud to share that Games Global has been shortlisted for four categories at the EGR B2B Awards 2025: ⭐ Employer of the Year ⭐ Slot Supplier...', hashtags: '#GamesGlobal, #EGRB2BAwards, #iGaming, #EGR2025, #InnovationInGaming', likes: 75, comments: 0, shares: 3 },
                 { content: '⚡ Another Gold Blitz™ hit has arrived! Say hello to Gold Blitz™ Fortunes from Fortune Factory Studios. 💥💰...', hashtags: '#GamesGlobal, #GoldBlitzFortunes, #FortuneFactoryStudios, #GoldBlitz', likes: 32, comments: 0, shares: 3 },
                { content: 'Spinnin’ Riches POWER COMBO™ by All For One brings the energy of Vegas to the reels, packed with features and dynamic gameplay...', hashtags: '#GamesGlobal, #SpinninRiches, #POWERCOMBO, #AllForOneStudios', likes: 24, comments: 1, shares: 1 }
            ]},
            { id: 'casinoGuru', name: 'Casino Guru', industry: '오퍼레이터', followers: 43398, growth: 29, posts: [
                 { content: 'Thanks to Casino Guru for putting together such a great Awards event in Malta last weekend. I appreciate the emphasis and importance placed on RG...', hashtags: '#betterinstitute, #gamblingrecovery', likes: 24, comments: 1, shares: 1 },
                { content: 'Such echoes speak for themselves, and we are thrilled to make this happen even once a year. You, our charming partners and brilliant-minded...', hashtags: '', likes: 1, comments: 0, shares: 0 },
                { content: '📢The past couple of months have been absolutely insane for our Complaint Team. ...466 cases were resolved, resulting in a total of $1,270,511 returned to players.', hashtags: '', likes: 2, comments: 0, shares: 0 },
            ]},
            { id: 'superbet', name: 'Superbet', industry: '오퍼레이터', followers: 751559, growth: 361, posts: [
                { content: 'Great feature story in Tech.eu, showcasing our tech-driven vision for an innovative and immersive digital entertainment ecosystem...', hashtags: '#GrowthJourney, #Techsylvania, #ProductInnovators, #TechDriven', likes: 131, comments: 3, shares: 8 },
                { content: 'Last weekend, Bucharest hosted a truly special event celebrating the remarkable career of Cristina Neagu, iconic female athlete and global handball star. 🌟...', hashtags: '#GalaNeagu, #SportsHub, #PassionforSports, #SuperbetRomania', likes: 93, comments: 0, shares: 3 },
                { content: 'Technology plays a central role in our journey of growth, enabling us to remain nimble and creative while shaping exceptional customer experiences...', hashtags: '#Techsylvania2025, #TechPowerhouse, #EntertainmentEcosystem, #ThoughtLeadership', likes: 49, comments: 0, shares: 1 }
            ]},
            { id: 'everymatrix', name: 'EveryMatrix', industry: '오퍼레이터', followers: 1424524, growth: 126, posts: [
                { content: 'Great insights from our own Casino Chief Commercial Officer, Marc Burroughes, featured in iGamingFuture! He breaks down how EveryMatrix is reshaping player retention...', hashtags: '#EngageSuite', likes: 27, comments: 0, shares: 0 },
                 { content: '💡 Get a glimpse of the people and culture behind EveryMatrix’s London launch and bold growth plans.', hashtags: '', likes: 18, comments: 0, shares: 0 },
                { content: 'A new era in horse racing begins! Nine months after acquiring FSB, we’re proud to unveil our first fully in-house horse racing product...', hashtags: '#OddsMatrix', likes: 14, comments: 0, shares: 0 }
            ]},
             { id: 'betsson', name: 'Betsson Group', industry: '어그리게이터', followers: 9214951, growth: 1073, posts: [
                { content: 'Wrapping up SBC Summit Malta with exciting news! Last night at the SBC Awards Europe 2025, we proudly took home \'𝗕𝗲𝘀𝘁 𝗔𝗳𝗳𝗶𝗹𝗶𝗮𝘁𝗲𝘀 𝗣𝗿𝗼𝗴𝗿𝗮𝗺𝗺𝗲\' and \'𝗖𝗮𝘀𝗶𝗻𝗼 𝗢𝗽𝗲𝗿𝗮𝘁𝗼𝗿 𝗼𝗳 𝘁𝗵𝗲 𝗬𝗲𝗮𝗿\'! 🏆🎰', hashtags: '#WeAreBetsson, #SBCSummitMalta, #SBCEuropeAwards, #ResponsibleGaming, #CasinoOperator', likes: 98, comments: 4, shares: 5 },
                { content: 'Mark your calendar for the upcoming iGX 𝐒𝐮𝐦𝐦𝐢𝐭 𝐢𝐧 𝐋𝐨𝐧𝐝𝐨𝐧, where our Head of PPC, Aline Oliveira, will take the stage.', hashtags: '#BetssonGroup, #WeAreBetsson, #iGX, #GamingIndustry', likes: 41, comments: 0, shares: 2 },
                { content: 'A few months ago, we sat down with Kenneth Attard, Enterprise Architect at Betsson Group, to learn more about his role...', hashtags: '#WeAreBetsson, #AWSCommunity, #AWSEvents, #DevOps, #CareerOpportunity', likes: 30, comments: 0, shares: 0 }
            ]},
             { id: 'playtech', name: 'Playtech', industry: '오퍼레이터', followers: 121347, growth: 538, posts: [
                 { content: 'This morning, we released our full-year results for 2024 and were delighted to report another strong year for Playtech. Mor Weizer, CEO, says, “2024 has been a landmark year for Playtech..."', hashtags: '#Playtech, #FY2024, #Financialresults', likes: 276, comments: 3, shares: 12 },
                { content: 'Today we announced our new partnership with MSC Cruises to deliver retail sports betting experiences to cruise ship guests.', hashtags: '#PlaytechSports, #MScCruises, #PlaytechNeon, #RetailSports, #Playtech, #CruiseShips', likes: 192, comments: 6, shares: 16 },
                { content: 'We are pleased to share our 2024 Annual Report, highlighting the strong progress we made in the year across our financial, strategic and operational goals.', hashtags: '#Playtech, #AnnualReport24', likes: 55, comments: 1, shares: 0 }
            ]},
            { id: 'evolution', name: 'Evolution', industry: 'CP', followers: 129288, growth: 990, posts: [
                 { content: 'We’re honoured to be recognised as the Best Digital Casino Supplier of the Year at the Global Gaming Awards Asia-Pacific 2025. 🏆', hashtags: '#Evolution, #GlobalGamingAwards2025, #DigitalCasinoSupplier', likes: 136, comments: 5, shares: 10 },
                { content: 'Last week Evolution participated at the biggest conference for Women in Tech in Europe. The Perspektywy Women in Tech Summit 2025...', hashtags: '#Evolution', likes: 85, comments: 8, shares: 1 },
                { content: 'Last week, on June 4th, we hosted a Meetup as part of our Global L&D program and to open our office to the local IT community.', hashtags: '', likes: 75, comments: 0, shares: 4 }
            ]},
        ];

        let followerChartInstance, industryChartInstance;
        let currentSort = { column: 'followers', order: 'desc' };

        document.addEventListener('DOMContentLoaded', () => {
            const processedData = companyData.map(c => ({...c, totalEngagement: c.posts.reduce((acc, p) => acc + p.likes + p.comments + p.shares, 0)}));
            setupFilters(processedData);
            
            document.getElementById('industryFilter').addEventListener('change', () => updateDashboard(processedData));
            document.getElementById('searchCompany').addEventListener('input', () => updateDashboard(processedData));
            document.getElementById('resetButton').addEventListener('click', () => {
                document.getElementById('industryFilter').value = 'All';
                document.getElementById('searchCompany').value = '';
                updateDashboard(processedData);
            });
            document.querySelectorAll('.table-sortable th').forEach(header => {
                if(!header.dataset.sort) return;
                header.addEventListener('click', () => {
                    const column = header.dataset.sort;
                    if (currentSort.column === column) {
                        currentSort.order = currentSort.order === 'asc' ? 'desc' : 'asc';
                    } else {
                        currentSort.column = column;
                        currentSort.order = 'desc';
                    }
                    updateDashboard(processedData);
                });
            });
            
            updateDashboard(processedData);
             window.addEventListener('resize', () => updateDashboard(processedData));
        });

        function setupFilters(data) {
            const industries = [...new Set(data.map(c => c.industry))].sort();
            const industryFilter = document.getElementById('industryFilter');
             // Clear previous options except the first one
            industryFilter.innerHTML = '<option value="All">전체</option>';
            industries.forEach(i => {
                const option = document.createElement('option');
                option.value = i;
                option.textContent = i;
                industryFilter.appendChild(option);
            });
        }
        
        function getFilteredData(data) {
            const selectedIndustry = document.getElementById('industryFilter').value;
            const searchTerm = document.getElementById('searchCompany').value.toLowerCase();
            
            return data
                .filter(c => selectedIndustry === 'All' || c.industry === selectedIndustry)
                .filter(c => c.name.toLowerCase().includes(searchTerm));
        }
        
        function sortData(data) {
             return [...data].sort((a, b) => {
                const valA = (typeof a[currentSort.column] === 'string') ? a[currentSort.column].toUpperCase() : a[currentSort.column];
                const valB = (typeof b[currentSort.column] === 'string') ? b[currentSort.column].toUpperCase() : b[currentSort.column];
                
                let comparison = 0;
                if (valA > valB) {
                    comparison = 1;
                } else if (valA < valB) {
                    comparison = -1;
                }
                
                return currentSort.order === 'asc' ? comparison : comparison * -1;
            });
        }

        function updateDashboard(data) {
            const filteredData = getFilteredData(data);
            const sortedData = sortData(filteredData);
            
            updateKPIs(filteredData);
            updateFollowerChart(sortData(filteredData, {column: 'followers', order: 'desc'}));
            updateIndustryChart(filteredData);
            updateCompanyTable(sortedData);
            updateSortIndicators();
        }
        
        function updateKPIs(data) {
            document.getElementById('totalCompanies').textContent = data.length;
            const totalFollowers = data.reduce((acc, c) => acc + c.followers, 0);
            document.getElementById('totalFollowers').textContent = totalFollowers > 10000 ? `${(totalFollowers/1000).toFixed(1)}k` : totalFollowers.toLocaleString('ko-KR');

            if (data.length > 0) {
                 const topGrowth = [...data].sort((a, b) => b.growth - a.growth)[0];
                 document.getElementById('topGrowthCompany').textContent = `${topGrowth.name} (+${topGrowth.growth.toLocaleString()})`;
                 
                 const topPost = data.map(c => ({
                     company: c.name,
                     topPostEngagement: Math.max(...c.posts.map(p => p.likes + p.comments + p.shares))
                 })).sort((a,b) => b.topPostEngagement - a.topPostEngagement)[0];
                 document.getElementById('topPostCompany').textContent = `${topPost.company} (${topPost.topPostEngagement.toLocaleString()} 참여)`;
            } else {
                document.getElementById('topGrowthCompany').textContent = '-';
                document.getElementById('topPostCompany').textContent = '-';
            }
        }
        
        function updateFollowerChart(data) {
            const ctx = document.getElementById('followerChart').getContext('2d');
            const isMobile = window.innerWidth < 768;
            const itemsToShow = isMobile ? 8 : 15;
            const chartData = data.slice(0, itemsToShow);
            
            if (followerChartInstance) followerChartInstance.destroy();
            
            followerChartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: chartData.map(c => c.name),
                    datasets: [{
                        label: '팔로워 수',
                        data: chartData.map(c => c.followers),
                        backgroundColor: '#3b82f6',
                        borderColor: '#1d4ed8',
                        borderWidth: 1
                    }]
                },
                options: {
                    indexAxis: 'y',
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: { 
                        legend: { display: false }
                    },
                    scales: {
                        x: {
                            ticks: {
                                font: { size: isMobile ? 8 : 12 },
                                callback: function(value) {
                                    if (value >= 1000000) return (value / 1000000) + 'M';
                                    if (value >= 1000) return (value / 1000) + 'k';
                                    return value;
                                }
                            }
                        },
                        y: {
                            ticks: {
                                font: { size: isMobile ? 9 : 12 }
                            }
                        }
                    }
                }
            });
        }
        
        function updateIndustryChart(data) {
            const ctx = document.getElementById('industryChart').getContext('2d');
            const isMobile = window.innerWidth < 768;
            const industryCounts = data.reduce((acc, c) => {
                acc[c.industry] = (acc[c.industry] || 0) + 1;
                return acc;
            }, {});
            
            if (industryChartInstance) industryChartInstance.destroy();
            
            industryChartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: Object.keys(industryCounts),
                    datasets: [{
                        data: Object.values(industryCounts),
                        backgroundColor: ['#10b981', '#ef4444', '#f97316', '#8b5cf6', '#3b82f6', '#ec4899', '#64748b'],
                        borderColor: '#f4f7f6',
                        borderWidth: 4
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { 
                            position: 'bottom',
                            labels: {
                                font: { size: isMobile ? 10 : 12 },
                                boxWidth: isMobile ? 15 : 20,
                                padding: isMobile ? 10 : 20
                            }
                        }
                    }
                }
            });
        }

        function updateCompanyTable(data) {
            const tableBody = document.getElementById('companyTableBody');
            tableBody.innerHTML = '';
            if (data.length === 0) {
                const row = tableBody.insertRow();
                row.innerHTML = `<td colspan="4" class="text-center p-8 text-gray-500">표시할 데이터가 없습니다.</td>`;
                return;
            }
            
            data.forEach(company => {
                const row = document.createElement('tr');
                row.className = 'bg-white border-b hover:bg-gray-50 cursor-pointer';
                row.dataset.companyId = company.id;
                
                const growthColor = company.growth >= 0 ? 'text-green-600' : 'text-red-600';
                const growthSign = company.growth >= 0 ? '▲' : '▼';
                
                row.innerHTML = `
                    <td class="p-2 text-center">
                        <span class="toggle-btn inline-block text-blue-500 text-xl font-bold">▸</span>
                    </td>
                    <td class="px-3 py-3 md:px-6 font-medium text-gray-900 whitespace-nowrap">${company.name}</td>
                    <td class="px-3 py-3 md:px-6">${company.followers.toLocaleString()}</td>
                    <td class="px-3 py-3 md:px-6 ${growthColor}">${growthSign} ${Math.abs(company.growth).toLocaleString()}</td>
                `;
                
                const detailRow = document.createElement('tr');
                detailRow.className = 'post-details';
                detailRow.id = `posts-${company.id}`;
                
                let postsHtml = company.posts.map(post => {
                    const totalEngagement = post.likes + post.comments + post.shares;
                    return `
                        <div class="border-t border-gray-200 p-3">
                            <p class="text-gray-700 mb-2 whitespace-pre-wrap text-xs">${post.content}</p>
                            <p class="text-xs text-blue-700 font-semibold mb-2 truncate">${post.hashtags}</p>
                            <div class="flex items-center text-xs text-gray-500 space-x-3">
                                <span>👍${post.likes}</span>
                                <span>💬${post.comments}</span>
                                <span>🔁${post.shares}</span>
                                <span class="font-bold">✨총 ${totalEngagement}</span>
                            </div>
                        </div>
                    `;
                }).join('');
                
                detailRow.innerHTML = `<td colspan="4" class="p-0"><div class="p-2 bg-blue-50/50">${postsHtml}</div></td>`;
                
                tableBody.appendChild(row);
                tableBody.appendChild(detailRow);

                row.addEventListener('click', () => {
                     row.classList.toggle('open');
                    detailRow.classList.toggle('open');
                });
            });
        }
        
        function updateSortIndicators() {
            document.querySelectorAll('.table-sortable th').forEach(header => {
                const indicator = header.querySelector('.sort-indicator');
                if(!indicator) return;
                
                if (header.dataset.sort === currentSort.column) {
                    indicator.textContent = currentSort.order === 'asc' ? '▲' : '▼';
                    indicator.style.opacity = '1';
                } else {
                    indicator.textContent = '▼';
                    indicator.style.opacity = '0.3';
                }
            });
        }
    </script>
</body>
</html>
