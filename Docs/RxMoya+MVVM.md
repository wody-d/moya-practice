# RxMoya + MVVM

๐ *GithubAPI๋ฅผ ์ฌ์ฉํ์ต๋๋ค.*

๐ *RxMoya+MVVM ์์ ์๋๋ค.*



- `GithubAPI.swift`

  TargetType์ ์ฑํํ ์ด๊ฑฐ์ฒด๋ฅผ ๋ง๋ค์ด์ค๋๋ค. (repository์ issue๋ฅผ ๋ฐ์์ฌ ๊ฒฝ๋ก 2๊ฐ๋ฅผ ๊ตฌํํด์ฃผ์์ต๋๋ค.)

```swift

enum GithubAPI {
  case getRepositorByUserID(String)
  case getIssueInOrganization(String)
}

typealias Method = Moya.Method

extension GithubAPI: TargetType {
  var baseURL: URL {
    return URL(string: "https://api.github.com")!
  }
  // MARK: - ๊ฒฝ๋ก
  var path: String {
    switch self {
    case .getRepositorByUserID(let userID):
      return "/users/\(userID)/repos"
    case .getIssueInOrganization(let organization):
      return "/orgs/\(organization)/issues"
    }
  }
  // MARK: - REST API ํ์
  var method: Method {
    switch self {
    case .getRepositorByUserID:
      return .get
    case .getIssueInOrganization:
      return .get
    }
  }
  // MARK: - ๋ฐ์ดํฐ
  var task: Task {
    switch self {
    case .getRepositorByUserID:
      return .requestPlain
    case .getIssueInOrganization:
      return .requestPlain
    }
  }
  var headers: [String : String]? {
    [
      "Content-Type": "application/json",
      "Authorization": "" // TODO: ํ ํฐ ์ฝ์
    ]
  }
}
```



- `ViewModel.swift`

  `requestRepository`ํจ์๋ฅผ VC์์ ๋ถ๋ฌ์ ๋ณด์ด์ง ์๋ `_requestRepository` ํจ์๋ฅผ ์ด์ฉํด ์์ฒญ์ ๋ฐ์์จ ํ `repositoryDatas`์ ์ ๋ฌํด์ค๋๋ค. 

```swift

class ViewModel: BaseViewModel {
    let repositoryDatas = BehaviorRelay<[RepositoryModel]>(value: [])
    let provider = MoyaProvider<GithubAPI>() 
    func requestRepository(userID: String) {
        _requestRepository(userID: userID)
            .subscribe(onSuccess: { [weak self] in
                self?.repositoryDatas.accept($0)
            })
            .disposed(by: disposeBag)
    }
    private func _requestRepository(userID: String) -> Single<[RepositoryModel]> {
        return provider.rx.request(.getRepositorByUserID(userID))
            .flatMap { response in
                do {
                    return .just(try response.map([RepositoryModel].self))
                } catch(let err) {
                    print(err.localizedDescription)
                    fatalError()
                }
            }
    }
}
```



- `ViewController.swift`

  tableView์ dataSource๋ `repositoryDatas`๋ฅผ ๊ตฌ๋ํด๋ก๋๋ค. repository ์์ฒญ ๋ฒํผ์ ๋๋ฅด๋ฉด ์์์ ๊ตฌํํ์๋ `requestRepository`ํจ์๋ฅผ ์ด์ฉํ์ฌ ์์ฒญ์ ๋ณด๋๋๋ค. `repositoryDatas`๊ฐ ์๋ฐ์ดํธ๊ฐ ๋๋ฉด tableView๋ ์๋ฐ์ดํธ๊ฐ ๋ฉ๋๋ค.

```swift
class ViewController: BaseViewController<ViewModel> {
    let disposeBag = DisposeBag()
    @IBOutlet weak var tableView: UITableView!
    @IBOutlet weak var repositoryButton: UIButton!
    @IBOutlet weak var issueButton: UIButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        setupRepositoryButton()
        setupTableView()
    }
    
    private func setupRepositoryButton() {
        repositoryButton.rx.tap
            .bind(onNext: { [weak self] _ in
                guard let self = self else { return }
                self.viewModel.requestRepository(userID: "wody27")
            })
            .disposed(by: disposeBag)
    }
    
    private func setupTableView() {
        viewModel.repositoryDatas
            .bind(to: tableView.rx.items(cellIdentifier: "TableViewCell",
                                         cellType: TableViewCell.self)) { row, cellModel, cell in
                cell.nameLabel.text = cellModel.name
            }.disposed(by: disposeBag)
        tableView.rx.setDelegate(self).disposed(by: disposeBag)
        
    }
}
```



### [+์ถ๊ฐ](https://github.com/wody27/moya-practice/blob/main/Docs/+more.md)

- ViewController์ ViewModel ์ฃผ์ํ๊ธฐ (+ BaseViewModel, BaseViewController) 

- rx๋ฅผ ์ด์ฉํ tableView datasource

