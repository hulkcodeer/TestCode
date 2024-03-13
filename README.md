# 신의 훈수 TEST CODE

//
//  godsadviceTests.swift
//  godsadviceTests
//
//  Created by Hyun Jin Park on 2023/11/16.
//

import XCTest
import Combine
@testable import godsadvice

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

class MyPageViewBlocTests: XCTestCase {
    var bloc: MyPageViewBloc!
    var mockAPI: MockMyPageAPI!
    var cancelables: Set<AnyCancellable> = []

    override func setUpWithError() throws {
        super.setUp()
                
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
        let expectation = XCTestExpectation(description: "State should be updated")

        bloc.$state
            .sink { state in
                if state.displayProfileImgUrl != "" {
                    expectation.fulfill()
                }
            }
            .store(in: &cancelables)
        
        bloc.event(.fetchMyPageInfo)
        
        wait(for: [expectation], timeout: 5.0)
    }
}
