[gemini-code-1779075079413.html](https://github.com/user-attachments/files/27942477/gemini-code-1779075079413.html)
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>카카오맵 장소 검색 서비스</title>
    <style>
        /* 웹페이지 전체 레이아웃 초기화 */
        body {
            margin: 0;
            padding: 0;
            font-family: 'Malgun Gothic', dotum, sans-serif;
            display: flex;
            height: 100vh;
        }

        /* 왼쪽 사이드바 스타일 (검색창 + 검색 리스트) */
        #sidebar {
            width: 350px;
            height: 100%;
            background-color: #f7f7f7;
            display: flex;
            flex-direction: column;
            border-right: 1px solid #e0e0e0;
            box-sizing: border-box;
            z-index: 10;
        }

        /* 검색창 영역 */
        #search-box {
            padding: 20px;
            background-color: #ffffff;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            display: flex;
            gap: 10px;
        }

        #search-input {
            flex: 1;
            padding: 10px;
            font-size: 14px;
            border: 1px solid #ccc;
            border-radius: 4px;
            outline: none;
        }

        #search-btn {
            padding: 10px 15px;
            font-size: 14px;
            background-color: #fee500; /* 카카오 시그니처 옐로우 */
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
        }

        #search-btn:hover {
            background-color: #fada0a;
        }

        /* 검색 결과 리스트 영역 */
        #places-list {
            list-style: none;
            padding: 0;
            margin: 0;
            overflow-y: auto; /* 리스트가 길어지면 스크롤 생성 */
            flex: 1;
        }

        #places-list li {
            padding: 15px;
            border-bottom: 1px solid #eeeeee;
            background-color: #ffffff;
            cursor: pointer;
            transition: background-color 0.2s;
        }

        #places-list li:hover {
            background-color: #f5f9ff;
        }

        .place-name {
            font-size: 16px;
            font-weight: bold;
            color: #333333;
            margin-bottom: 5px;
        }

        .place-address {
            font-size: 13px;
            color: #666666;
            margin-bottom: 3px;
        }

        .place-phone {
            font-size: 12px;
            color: #008b8b;
        }

        /* 오른쪽 지도 영역 */
        #map {
            flex: 1;
            height: 100%;
        }
    </style>
</head>
<body>

    <!-- 왼쪽 사이드바 -->
    <div id="sidebar">
        <div id="search-box">
            <input type="text" id="search-input" placeholder="원하는 키워드(예: 맛집, 카페)" value="이태원 맛집">
            <button id="search-btn">검색</button>
        </div>
        <ul id="places-list"></ul>
    </div>

    <!-- 오른쪽 지도 -->
    <div id="map"></div>

    <!-- 카카오맵 API 로드 (장소 검색을 위해 services 라이브러리 추가 필수) -->
    <script type="text/javascript" src="//dapi.kakao.com/v2/maps/sdk.js?appkey=YOUR_KAKAO_APP_KEY_HERE&libraries=services"></script>
    
    <script>
        // 1. 지도 초기화
        const mapContainer = document.getElementById('map');
        const mapOption = {
            center: new kakao.maps.LatLng(37.566826, 126.978656), // 초기 중심 좌표 (서울시청)
            level: 3 // 지도 확대 레벨
        };
        const map = new kakao.maps.Map(mapContainer, mapOption);

        // 2. 카카오 로컬 장소 검색 서비스 객체 생성
        const placeService = new kakao.maps.services.Places();

        // 3. 팝업창(InfoWindow) 객체 생성
        const infoWindow = new kakao.maps.InfoWindow({ zIndex: 1 });

        // 4. 데이터 관리를 위한 배열 생성
        let markers = []; // 생성된 마커들을 저장할 배열

        // 5. 이벤트 리스너 등록 (버튼 클릭 및 엔터키 입력)
        document.getElementById('search-btn').addEventListener('click', searchPlaces);
        document.getElementById('search-input').addEventListener('keypress', function(event) {
            if (event.key === 'Enter') {
                searchPlaces();
            }
        });

        // 6. 키워드 검색 실행 함수
        function searchPlaces() {
            const keyword = document.getElementById('search-input').value.trim();

            if (!keyword) {
                alert('검색어를 입력해주세요!');
                return;
            }

            // 카카오 API로 키워드 검색 요청
            placeService.keywordSearch(keyword, placesSearchCallback);
        }

        // 7. 장소 검색 완료 후 호출되는 콜백 함수
        function placesSearchCallback(data, status, pagination) {
            if (status === kakao.maps.services.Status.OK) {
                // 새로운 검색을 위해 기존 마커와 리스트 지우기
                clearResults();
                
                // 검색 결과 지도 및 리스트에 표시
                displayResults(data);
            } else if (status === kakao.maps.services.Status.ZERO_RESULT) {
                alert('검색 결과가 존재하지 않습니다.');
            } else if (status === kakao.maps.services.Status.ERROR) {
                alert('검색 중 오류가 발생했습니다.');
            }
        }

        // 8. 검색 결과를 화면과 지도에 그리는 함수
        function displayResults(places) {
            const listContainer = document.getElementById('places-list');
            const bounds = new kakao.maps.LatLngBounds(); // 검색된 모든 마커를 포함할 지도 범위 객체

            places.forEach(function(place) {
                // 장소 좌표 생성
                const placePosition = new kakao.maps.LatLng(place.y, place.x);
                
                // 지도에 마커 생성 및 표시
                const marker = new kakao.maps.Marker({
                    position: placePosition,
                    map: map
                });
                markers.push(marker); // 추후 삭제를 위해 배열에 보관

                // 지도 범위 확장을 위해 마커 좌표 추가
                bounds.extend(placePosition);

                // 왼쪽 리스트에 넣을 HTML 아이템 엘리먼트 생성
                const listItem = document.createElement('li');
                
                // 도로명 주소가 있으면 사용하고, 없으면 지번 주소 사용
                const addressName = place.road_address_name ? place.road_address_name : place.address_name;
                const phoneName = place.phone ? place.phone : '번호 없음';

                listItem.innerHTML = `
                    <div class="place-name">${place.place_name}</div>
                    <div class="place-address">${addressName}</div>
                    <div class="place-phone">${phoneName}</div>
                `;

                // [클릭 이벤트 핸들러 고정] 클로저(Closure) 문제를 피하기 위해 즉시실행함수 형태로 묶음
                (function(currentMarker, currentPlace) {
                    // 지도 위 마커 클릭 시 인포윈도우 표시
                    kakao.maps.event.addListener(currentMarker, 'click', function() {
                        showPopup(currentMarker, currentPlace);
                    });

                    // 왼쪽 리스트 아이템 클릭 시 지도 이동 및 인포윈도우 표시
                    listItem.addEventListener('click', function() {
                        map.panTo(placePosition); // 해당 좌표로 부드럽게 화면 이동
                        showPopup(currentMarker, currentPlace);
                    });
                })(marker, place);

                // 생성된 아이템을 리스트에 추가
                listContainer.appendChild(listItem);
            });

            // 검색된 모든 장소가 한눈에 보이도록 지도 화면 범위 재설정
            map.setBounds(bounds);
        }

        // 9. 인포윈도우(팝업) 띄우기 함수
        function showPopup(marker, place) {
            const content = `
                <div style="padding:10px; min-width:150px; font-size:13px; line-height: 1.5;">
                    <strong style="display:block; margin-bottom:5px; color:#0056b3;">${place.place_name}</strong>
                    <span style="color:#666; font-size:11px;">${place.category_group_name ? place.category_group_name : '장소'}</span>
                </div>
            `;
            infoWindow.setContent(content);
            infoWindow.open(map, marker);
        }

        // 10. 새로 검색 시 기존 데이터 초기화 함수
        function clearResults() {
            // 지도 위 모든 마커 제거
            markers.forEach(function(marker) {
                marker.setMap(null);
            });
            markers = []; // 마커 배열 비우기

            // 왼쪽 검색 리스트 비우기
            const listContainer = document.getElementById('places-list');
            listContainer.innerHTML = '';

            // 열려있는 인포윈도우 닫기
            infoWindow.close();
        }

        // 페이지 로드 시 첫 검색 자동 실행 (초기 더미 검색)
        searchPlaces();
    </script>
</body>
</html>
