---
title:  "Protocol Oriented Loading Views"
date:   2017-08-02 08:58:00 -0800
categories: swift protocols pop
---


```
protocol LoadingPresenter {

    func showLoadingView()
    func hideLoadingView()
}
```

```
protocol LoadingNavbarPresenter: LoadingPresenter {

    var navigationItem: UINavigationItem { get }
}
```

```
extension LoadingNavbarPresenter {

    func showLoadingView() {
        let spinner = UIActivityIndicatorView(activityIndicatorStyle: .white)
        let barButton = UIBarButtonItem(customView: spinner)
        self.navigationItem.rightBarButtonItem = barButton
        spinner.startAnimating()
    }

    func hideLoadingView() {
        guard let spinner = self.navigationItem.rightBarButtonItem?.customView as? UIActivityIndicatorView else { return }
        spinner.stopAnimating()
        self.navigationItem.rightBarButtonItem = nil
    }
}
```
