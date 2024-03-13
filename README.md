# 신의 훈수 TEST CODE
# 테스트 코드 설계
# ViewModel 관련 로직만 작성
# Mock API설정을 테스트 코드 단계에서 할지?...아니면 클래스에서 제공할지 고민중...
# Mock 데이터는 실제 JSON을 담을지, 내가 지정한 데이터를 담을지도 고민중...
# Repository단계 테스트가 필요할까?...

# MockMyPageAPI 설정
```swift
class MockMyPageAPI: MyPageAPI {
    var myPageInfoPublisher: AnyPublisher<MyPageModel, APIError>
    var myPageHistoryMetaInfoPublisher: AnyPublisher<MyPageHistoryMetaModel, APIError>
    
    init(myPageInfoPublisher: AnyPublisher<MyPageModel, APIError>,
         myPageHistoryMetaInfoPublisher: AnyPublisher<MyPageHistoryMetaModel, APIError>) {
        self.myPageInfoPublisher = myPageInfoPublisher
        self.myPageHistoryMetaInfoPublisher = myPageHistoryMetaInfoPublisher
    }
    
    func fetchMyPageInfo() -> AnyPublisher<MyPageModel, APIError> {
        return myPageInfoPublisher
    }
    
    func fetchMyPageHistoryMetaInfo() -> AnyPublisher<MyPageHistoryMetaModel, APIError> {
        return myPageHistoryMetaInfoPublisher
    }
}

# MockMyPageAPI TEST CODE
class MyPageViewBlocTests: XCTestCase {
    var bloc: MyPageViewBloc!
    var mockAPI: MockMyPageAPI!
    var cancelables: Set<AnyCancellable> = []

    override func setUpWithError() throws {
        super.setUp()
        // Given: 테스트 환경 설정
        let myPageInfoPublisher = Just(MyPageModel())
            .setFailureType(to: APIError.self)
            .eraseToAnyPublisher()
        
        let myPageHistoryMetaInfoPublisher = Just(MyPageHistoryMetaModel())
            .setFailureType(to: APIError.self)
            .eraseToAnyPublisher()
        
        mockAPI = MockMyPageAPI(myPageInfoPublisher: myPageInfoPublisher,
                                myPageHistoryMetaInfoPublisher: myPageHistoryMetaInfoPublisher)
        
        bloc = MyPageViewBloc(repository: mockAPI)
    }

    override func tearDownWithError() throws {
        bloc = nil
        mockAPI = nil
        cancelables.removeAll()
        super.tearDown()
    }

    func testFetchMyPageInfoUpdatesState() {
        // Given: 테스트 상태 설정
        let expectation = XCTestExpectation(description: "State should be updated")

        // When: 이벤트 실행
        bloc.event(.fetchMyPageInfo)

        // Then: 결과 검증
        bloc.$state
            .sink { state in
                if state.displayProfileImgUrl != "" {
                    expectation.fulfill()
                }
            }
            .store(in: &cancelables)

        wait(for: [expectation], timeout: 3.0)
    }
}
