- 코드
```java
@Component
public class RedisGeo {
    private final GeoOperations<String, String> geoOperations;
    public final static String GEO_KEY = "places";

    public RedisGeo(RedisTemplate<String, String> redisTemplate) {
        this.geoOperations = redisTemplate.opsForGeo();
    }

    public void addLocation(Long id, double longitude, double latitude) {
        geoOperations.add(GEO_KEY, new Point(longitude, latitude), id.toString());

    }

    public List<String> findByRadius(String key, double longitude, double latitude, double radius) {
        Circle circle = new Circle(new Point(longitude, latitude), new Distance(radius, Metrics.METERS));
        GeoResults<RedisGeoCommands.GeoLocation<String>> results = geoOperations.radius(key, circle);
        return results.getContent().stream()
                .map(result -> result.getContent().getName())
                .collect(Collectors.toList());
    }
}
```
```java
@Component
@RequiredArgsConstructor
public class KakaoAPIStoreInfoRedisSaver {

    private final PlaceRedisRepository placeRedisRepository;
    private final KakaoStoreAPI kakaoAPIRequest;
    private final RedisGeo redisGeo;

    // 2차 계획 (현재 미사용):
    // KakaoStoreAPIRequest로 결과 값 받아 Place 정보(Redis)와 좌표 정보(RedisGeo) 캐싱 후 지도에서 움직일때 움직인 범위의 좌표까지는 캐싱된 정보로 반환.
    // -> 검색했던 좌표 범위 추적하여 캐싱 구현 어려워, 3차 계획으로 변경됨
    // (3차 계획: tools/KakaoStoreInfoClient에 내역 위치)

    public void saveToRedisWithKeyWord(String query, double x, double y, int radius) {
        saveStoreInfoToRedis(SearchType.KEYWORD, query, x, y, radius);
    }

    public void saveToRedisWithCategory(SearchCategory category, double x, double y, int radius) {
        saveStoreInfoToRedis(SearchType.CATEGORY, category.toString(), x, y, radius);
    }

    private void saveStoreInfoToRedis(SearchType type, String query, double x, double y, int radius) {
        String response = kakaoAPIRequest.getStoreInfo(type, query, x, y, radius);
        List<Place> list = parsePlaceInfo(response);

        if (list.isEmpty()) return;
        for (Place place : list) {
            placeRedisRepository.save(PlaceRedis.fromKakao(place));
            redisGeo.addLocation(place.getId(), place.getX(), place.getY());
        }
    }


    private List<Place> parsePlaceInfo(String response) {
        try {
            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode rootNode = objectMapper.readTree(response);

            List<Place> places = new ArrayList<>();
            for (JsonNode document : rootNode) {
                Place place = Place.fromJsonNode(document);
                places.add(place);
            }
            return places;
        } catch (JsonProcessingException e) {
            throw new RuntimeException("Place로 변환 중 에러.");
        }
    }

}

```
```java
@Service
@RequiredArgsConstructor
@Slf4j
public class PlaceSearchServiceRedis {

    private final PlaceRedisRepository placeRedisRepository;
    private final KakaoAPIStoreInfoRedisSaver storeInfoSaver;
    private final RedisGeo redisGeo;


    public List<PlaceRedis> searchPlaceWithKeyword(String query, double x, double y, int radius) {
        storeInfoSaver.saveToRedisWithKeyWord(query, x, y, radius);

        return redisGeo.findByRadius(GEO_KEY, x, y, radius).stream()
                .map(placeId -> {
                    return placeRedisRepository.findById(placeId)
                            .orElseThrow(() -> new PlaceException(NO_PLACE_INFO));
                })
                .toList();
    }

    public List<PlaceRedis> searchWithCategory(SearchCategory category, double x, double y, int radius) {
        storeInfoSaver.saveToRedisWithCategory(category, x, y, radius);
        return redisGeo.findByRadius(GEO_KEY, x, y, radius).stream()
                .map(placeId -> {
                    return placeRedisRepository.findById(placeId)
                            .orElseThrow(() -> new PlaceException(NO_PLACE_INFO));
                })
                .toList();
    }

}
```
- 테스트
```java
@ExtendWith(MockitoExtension.class)
class RedisGeoTest {

    @Mock
    private RedisTemplate<String, String> redisTemplate;

    @Mock
    private GeoOperations<String, String> geoOperations;

    @InjectMocks
    private RedisGeo sut;

    @BeforeEach
    void setUp() {
        given(redisTemplate.opsForGeo()).willReturn(geoOperations);
        sut = new RedisGeo(redisTemplate);
    }

    @Test
    void addLocation() {
        // given
        Long id = 1L;
        double longitude = 126.9779692;
        double latitude = 37.566535;

        // when
        sut.addLocation(id, longitude, latitude);

        // then
        verify(geoOperations).add(eq(GEO_KEY), eq(new Point(longitude, latitude)), eq(id.toString()));
    }

    @Test
    void findByRadius() {
        // given
        Long id = 1L;
        double longitude = 126.9779692;
        double latitude = 37.566535;
        double radius = 100;

        RedisGeoCommands.GeoLocation<String> location1 = mock(RedisGeoCommands.GeoLocation.class);
        given(location1.getName()).willReturn("place:1");

        RedisGeoCommands.GeoLocation<String> location2 = mock(RedisGeoCommands.GeoLocation.class);
        given(location2.getName()).willReturn("place:2");

        GeoResults<RedisGeoCommands.GeoLocation<String>> geoResults =
                new GeoResults<>(Arrays.asList(
                        new GeoResult<>(location1, new Distance(100, Metrics.METERS)),
                        new GeoResult<>(location2, new Distance(200, Metrics.METERS))
                ));

        given(geoOperations.radius(anyString(), any(Circle.class))).willReturn(geoResults);

        //when
        List<String> returned = sut.findByRadius(GEO_KEY, longitude, latitude, radius);

        //then
        assertEquals(2, returned.size());
        assertEquals("place:1", returned.get(0));
        assertEquals("place:2", returned.get(1));
        verify(geoOperations).radius(eq(GEO_KEY), eq(new Circle(new Point(longitude, latitude), new Distance(100, Metrics.METERS))));
    }
}
```
```java
@ExtendWith(MockitoExtension.class)
@Deprecated
class KakaoAPIStoreInfoRedisSaverTest {

    @InjectMocks
    KakaoAPIStoreInfoRedisSaver sut;

    @Mock
    private KakaoStoreAPI kakaoAPIRequest;

    @Mock
    private PlaceRedisRepository placeRedisRepository;

    @Mock
    private RedisGeo redisGeo;

    private String response;

    @BeforeEach
    void setUp() {
        response =
                "[{\"address_name\":\"서울 중구 명동2가 25-36\",\"category_group_code\":\"FD6\",\"category_group_name\":\"음식점\",\"category_name\":\"음식점 > 분식\",\"distance\":\"0\",\"id\":\"10332413\",\"phone\":\"02-776-5348\",\"place_name\":\"명동교자 본점\",\"place_url\":\"http://place.map.kakao.com/10332413\",\"road_address_name\":\"서울 중구 명동10길 29\",\"x\":\"126.98561429978552\",\"y\":\"37.56255453417897\"}," +
                        "{\"address_name\":\"서울 중구 명동2가 4-2\",\"category_group_code\":\"FD6\",\"category_group_name\":\"음식점\",\"category_name\":\"음식점 > 한식\",\"distance\":\"42\",\"id\":\"26853115\",\"phone\":\"02-3789-9292\",\"place_name\":\"순남시래기 명동직영점\",\"place_url\":\"http://place.map.kakao.com/26853115\",\"road_address_name\":\"서울 중구 명동10길 35-20\",\"x\":\"126.985784007806\",\"y\":\"37.5629131513515\"}]";
    }

    @Test
    @DisplayName("Redis 저장 - 키워드 저장 성공")
    void saveToRedisWithKeyWord() {
        //given
        String query = "맛집";
        double x = 126.98561429978552;
        double y = 37.56255453417897;
        int radius = 50;
        given(kakaoAPIRequest.getStoreInfo(any(SearchType.class), anyString(), anyDouble(), anyDouble(), anyInt())).willReturn(response);

        //when
        sut.saveToRedisWithKeyWord(query, x, y, radius);

        //then
        verify(kakaoAPIRequest).getStoreInfo(SearchType.KEYWORD, "맛집", x, y, radius);
        verify(placeRedisRepository, times(2)).save(any());
        verify(redisGeo, times(2)).addLocation(anyLong(), anyDouble(), anyDouble());
    }

    @Test
    @DisplayName("Redis 저장 - 카테고리 저장 성공")
    void saveToRedisWithCategory() {
        //given
        double x = 126.98561429978552;
        double y = 37.56255453417897;
        int radius = 50;
        given(kakaoAPIRequest.getStoreInfo(any(SearchType.class), anyString(), anyDouble(), anyDouble(), anyInt())).willReturn(response);

        //when
        sut.saveToRedisWithCategory(SearchCategory.FD6, x, y, radius);

        //then
        verify(kakaoAPIRequest).getStoreInfo(SearchType.CATEGORY, SearchCategory.FD6.name(), x, y, radius);
        verify(placeRedisRepository, times(2)).save(any());
        verify(redisGeo, times(2)).addLocation(anyLong(), anyDouble(), anyDouble());
    }
}
```
```java
@ExtendWith(MockitoExtension.class)
@Deprecated
class SearchPlaceServiceRedisTest {

    @InjectMocks
    private PlaceSearchServiceRedis sut;

    @Mock
    private KakaoAPIStoreInfoRedisSaver storeInfoSaver;

    @Mock
    private RedisGeo redisGeo;

    @Mock
    private PlaceRedisRepository placeRedisRepository;

    //Redis에 저장
    @Test
    @DisplayName("Redis 저장 - 키워드 검색, 저장 성공")
    void searchPlaceWithKeyword_redis() {
        //given
        String query = "맛집";
        double x = 126.98561429978552;
        double y = 37.56255453417897;
        int radius = 50;

        //List<String> placeIds = Arrays.asList("place1");
        List<String> placeIds = mock();
        when(placeIds.stream()).thenReturn(Stream.of("1"));
        PlaceRedis placeRedisDTO = mock(PlaceRedis.class);

        doNothing().when(storeInfoSaver).saveToRedisWithKeyWord(query, x, y, radius);
        given(redisGeo.findByRadius(GEO_KEY, x, y, radius)).willReturn(placeIds);
        when(placeRedisRepository.findById(anyString())).thenReturn(Optional.of(placeRedisDTO));

        //when
        List<PlaceRedis> returnedPlacesRedis = sut.searchPlaceWithKeyword(query, x, y, radius);

        //then
        assertEquals(placeRedisDTO, returnedPlacesRedis.get(0));
    }

    @Test
    @DisplayName("Redis 저장 - 키워드 검색, 가게 정보 없음")
    void searchPlaceWithKeyword_redis_NoPlaceException() {
        //given
        String query = "맛집";
        double x = 126.98561429978552;
        double y = 37.56255453417897;
        int radius = 50;

        List<String> placeIds = mock();
        when(placeIds.stream()).thenReturn(Stream.of("1"));

        doNothing().when(storeInfoSaver).saveToRedisWithKeyWord(query, x, y, radius);
        given(redisGeo.findByRadius(GEO_KEY, x, y, radius)).willReturn(placeIds);
        when(placeRedisRepository.findById(anyString())).thenReturn(Optional.empty());

        //when
        Exception exception = assertThrows(PlaceException.class, () -> {
            sut.searchPlaceWithKeyword(query, x, y, radius);
        });

        //then
        assertEquals(PlaceException.class, exception.getClass());
        assertEquals("가게 정보가 없습니다.", exception.getMessage());
    }

    @Test
    @DisplayName("Redis 저장 - 카테고리 검색, 저장 성공")
    void searchWithCategory_redis() {
        //given
        double x = 126.98561429978552;
        double y = 37.56255453417897;
        int radius = 50;

        List<String> placeIds = mock();
        when(placeIds.stream()).thenReturn(Stream.of("1"));
        PlaceRedis placeRedisDTO = mock(PlaceRedis.class);

        doNothing().when(storeInfoSaver).saveToRedisWithCategory(SearchCategory.FD6, x, y, radius);
        given(redisGeo.findByRadius(GEO_KEY, x, y, radius)).willReturn(placeIds);
        when(placeRedisRepository.findById(anyString())).thenReturn(Optional.of(placeRedisDTO));

        //when
        List<PlaceRedis> returnedPlacesRedis = sut.searchWithCategory(SearchCategory.FD6, x, y, radius);

        //then
        assertEquals(placeRedisDTO, returnedPlacesRedis.get(0));
    }

    @Test
    @DisplayName("Redis 저장 - 카테고리 검색, 가게 정보 없음")
    void searchWithCategory_redis_NoPlaceException() {
        //given
        double x = 126.98561429978552;
        double y = 37.56255453417897;
        int radius = 50;

        List<String> placeIds = mock();
        when(placeIds.stream()).thenReturn(Stream.of("1"));

        doNothing().when(storeInfoSaver).saveToRedisWithCategory(SearchCategory.FD6, x, y, radius);
        given(redisGeo.findByRadius(GEO_KEY, x, y, radius)).willReturn(placeIds);
        when(placeRedisRepository.findById(anyString())).thenReturn(Optional.empty());

        //when
        Exception exception = assertThrows(PlaceException.class, () -> {
            sut.searchWithCategory(SearchCategory.FD6, x, y, radius);
        });

        //then
        assertEquals(PlaceException.class, exception.getClass());
        assertEquals("가게 정보가 없습니다.", exception.getMessage());
    }
}
```
