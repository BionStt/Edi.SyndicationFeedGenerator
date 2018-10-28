# Edi.SyndicationFeedGenerator
The .NET Core RSS/ATOM generator used in my blog system

## Example Usage

```
public async Task RefreshFeedFileForPostAsync(bool isAtom)
{
    Logger.LogInformation("Start refreshing feed for posts.");
    List<SimpleFeedItem> itemCollection = GetPostsAsRssFeedItems();

    var rw = new SyndicationFeedGenerator
    {
        HostUrl = AppSettings.BindingUrl,
        HeadTitle = AppSettings.RssTitle,
        HeadDescription = AppSettings.RssDescription,
        Copyright = AppSettings.RssCopyright,
        Generator = AppSettings.RssGeneratorName,
        FeedItemCollection = itemCollection,
        TrackBackUrl = AppSettings.BindingUrl,
        MaxContentLength = 0
    };

    if (isAtom)
    {
        Logger.LogInformation($"Writing ATOM file.");
        await rw.WriteAtom10FileAsync($@"{AppDomain.CurrentDomain.GetData(Constants.DataDirectory)}\feed\posts-atom.xml");
    }
    else
    {
        Logger.LogInformation($"Writing RSS file.");
        await rw.WriteRss20FileAsync($@"{AppDomain.CurrentDomain.GetData(Constants.DataDirectory)}\feed\posts.xml");
    }

    Logger.LogInformation("Finished writing feed for posts.");
}

private List<SimpleFeedItem> GetPostsAsRssFeedItems(Guid? categoryId = null)
{
    Logger.LogInformation($"{nameof(GetPostsAsRssFeedItems)} - {nameof(categoryId)}: {categoryId}");

    int? top = null;
    if (AppSettings.RssItemCount != 0)
    {
        top = AppSettings.RssItemCount;
    }

    var query = GetSubscribedPosts(categoryId, top);

    var items = query.Select(p => p.PostPublish.PubDateUtc != null ? new SimpleFeedItem
    {
        Id = p.Id.ToString(),
        Title = p.Title,
        PubDateUtc = p.PostPublish.PubDateUtc.Value,
        Description = p.ContentAbstract,
        Link = GetPostLink(p.PostPublish.PubDateUtc.Value, p.Slug),
        Author = Constants.FeedAuthorName,
        AuthorEmail = AppSettings.EmailSettings.AdminEmail,
        Categories = p.PostCategory.Select(pc => pc.Category.DisplayName).ToList()
    } : null);

    return items.ToList();
}
```
