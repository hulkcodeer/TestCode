# 신의 훈수 TEST CODE

# MockMyPageAPI 설정
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
        let myPageInfoPublisher = Just(MyPageModel(/* 적절한 테스트 데이터로 초기화 */))
            .setFailureType(to: APIError.self)
            .eraseToAnyPublisher()
        
        let myPageHistoryMetaInfoPublisher = Just(MyPageHistoryMetaModel(/* 적절한 테스트 데이터로 초기화 */))
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
                    expectation.fulfill()  // 예상된 상태 업데이트가 발생하면 테스트를 성공적으로 마침
                }
            }
            .store(in: &cancelables)

        wait(for: [expectation], timeout: 5.0)  // 비동기 작업의 완료를 대기
    }
}
