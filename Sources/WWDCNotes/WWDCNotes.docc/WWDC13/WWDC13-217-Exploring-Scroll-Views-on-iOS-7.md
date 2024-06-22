# Exploring Scroll Views on iOS 7

UIScrollView is an important building block for constructing iOS interfaces. Join us for a tour of how scroll views are used in new and interesting ways across iOS 7 to create stunning interactions. Learn tips and tricks for using scroll views to create immersive effects in your apps.

@Metadata {
   @TitleHeading("WWDC13")
   @PageKind(sampleCode)
   @CallToAction(url: "http://developer.apple.com/wwdc13/217", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(antonio081014)
   }
}



## Nested Scroll Views

At first, the session presents how to make nested scroll views working together by using a `UICollectionView` in `UIScrollView`, while each `UICollectionViewCell` has a `UIScrollView` as the subview.
Meanwhile, the `UICollectionView` is the subclass of `UIScrollView`.

```
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
     determine delta beyond catch point
     adjust parent contentOffset by delta
     translate child by delta
}
```

Key points in this part:
- When subview scroll, using a delegate call to scroll parent scrollview(set contentOffset).
- When subview scroll, avoiding double scroll by transforming itself, transform itself back when scroll ends.
- When `scrollViewWillEndDragging`, adjust the scrollview `contentOffset`, change ratio between sub-scrollview and parent-scrollview.

## Scrolling with UIKit Dynamics

In the second part, the session presents how to add special animation effect in the `UICollectionView` by customizing `UICollectionFlowLayout` by adding `UIDynamicBehavior` to `UICollectionViewLayoutAttributes`.

- Subclass `UICollectionViewFlowLayout`
- Create `UIDynamicAnimator`
- Create `UIAttachmentBehavior` for each item 
- Stretch the attachments when scrolling

```
- (BOOL)shouldInvalidateLayoutForBoundsChange:(CGRect)newBounds {
     CGFloat delta = newBounds.origin.y - self.collectionView.bounds.origin.y
     shift layout attribute positions by delta
     notify UIDynamicAnimator
}
```

## Memory Efficient Code

In the second part, when `prepareLayout` is called, all the behaviors of every cells will be created, initialized and attached to the cell. In this part, the behaviors will be added to the cell when the cell appear on the screen, and removed when the cell disappear from the screen, very efficient.