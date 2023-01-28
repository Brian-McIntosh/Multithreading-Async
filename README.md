# Multithreading-Async

- This causes the UI to freeze
- each thing happens sequentially one after the other
- the next step can't happen until the previous step finishes downloading
- The reason is b/c we're doing this on the main thread
- e.g. an algorithm or saving data to file, anything that can take time
```swift
@IBAction func getRandomImage(_ sender: Any) {
    let url = URL(string: "https://loremflickr.com/2000/2000")!
    let data = try! Data(contentsOf: url)
    // here's where it freezes
    let image = UIImage(data: data)
    self.imageView.image = image
}
```

- in iOS, we don't think about threads, we think about queues
- DispatchQueue is part of Grand Central Dispatch
```swift
@IBAction func getRandomImage2(_ sender: Any) {
    DispatchQueue.global(qos: .background).async {
        let url = URL(string: "https://loremflickr.com/2000/2000")!
        let data = try! Data(contentsOf: url)
        let image = UIImage(data: data)
        /*
         This will cause a crash!
         Trying to update UI from a background thread (queue)
         */
        self.imageView.image = image
    }
}
```
    
Fix the crash by moving the UI work to the main queue
```swift
@IBAction func getRandomImage3(_ sender: Any) {
    DispatchQueue.global(qos: .background).async {
        let url = URL(string: "https://loremflickr.com/2000/2000")!
        let data = try! Data(contentsOf: url)
        let image = UIImage(data: data)

        DispatchQueue.main.async {
            self.imageView.image = image
        }
    }
}
```
    
- FINALLY, we never download data this way! Use URLSession.
- This actually takes care of some of the multithreading for us.
```swift
@IBAction func getRandomImage4(_ sender: Any) {

    let url = URL(string: "https://loremflickr.com/2000/2000")!
    let task = URLSession.shared.downloadTask(with: url) { (localUrl, response, error) in

        let data = try! Data(contentsOf: localUrl!)
        let image = UIImage(data: data)

        DispatchQueue.main.async {
            self.imageView.image = image
        }
    }
    task.resume()
}
```
