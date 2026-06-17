# Simple Event Photo Gallery Tutorial with GitHub Pages

This tutorial shows how to publish event photos on a simple public website using **GitHub Pages** for the site and any **S3-compatible object storage** for the photos. The goal is to keep the setup as simple as possible so other organizers can copy it without learning a heavy platform or building a custom app.

## What this setup is

This approach uses only two parts:

- **GitHub Pages** hosts the website.
- **S3-compatible object storage** holds the photo files.

The site itself is just a static `index.html` file in a GitHub repository, and the images are loaded from public object URLs in the storage bucket. This keeps the repo small and avoids checking hundreds of large photo files into Git.[cite:208][cite:81]

## Why this approach was chosen

This workflow was chosen because it is:

- **Cheap**, since GitHub Pages is free for static sites and object storage is inexpensive.[cite:68][cite:81]
- **Simple**, because there is no database, app server, CMS, or photo platform to maintain.[cite:208]
- **Portable**, because any S3-compatible object storage can be used the same way.[cite:68][cite:187]
- **Easy to hand off**, because the site can remain a single HTML file that anyone on the team can edit.

## Decisions behind this tutorial

To keep the workflow easy for others to adopt, these decisions were made:

- Use **GitHub Pages** instead of a heavier hosting platform.[cite:89][cite:91]
- Use a repo named **`photos`** so the purpose is obvious.
- Keep the site as a plain static page instead of a custom app.
- Store photos in object storage, not in the Git repo.[cite:208]
- Make the photo bucket public for read access so photos can be embedded directly by URL.[cite:191][cite:92]
- Avoid extra build steps unless they are truly needed.

This is intentionally not the most advanced gallery setup. It is the one that is easiest to repeat.

## What you need before starting

Before starting, make sure you have:

- A GitHub organization or personal GitHub account.
- A custom domain or subdomain, such as `photos.example.org`.
- A bucket in any S3-compatible object storage.
- Event photos ready to upload.
- Permission to create DNS records for your domain.

This tutorial assumes you already know how to create a bucket and upload photos into it.

## Architecture

The complete setup looks like this:

1. Create a GitHub repo named `photos`.
2. Add a simple `index.html` file to that repo.
3. Enable GitHub Pages for the repo.[cite:89][cite:135]
4. Point a custom domain such as `photos.example.org` to GitHub Pages.[cite:81][cite:91]
5. Upload photos to object storage.
6. Make the photo objects public for read access.[cite:191][cite:92]
7. Reference those photo URLs in the `index.html` gallery.

## Step 1: Create the GitHub repo

Create a new repository named **`photos`** in your GitHub organization or personal account.

Recommended settings:

- Repository name: `photos`
- Visibility: Public
- Initialize with a README: optional
- Add `.gitignore`: not required
- License: optional

Using a simple repo name like `photos` makes the purpose clear and works nicely with a custom domain like `photos.example.org`.

## Step 2: Add a basic `index.html`

In the root of the repo, create a file named `index.html`.

Start with this minimal version:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Event Photos</title>
  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #111;
      color: #fff;
    }
    header {
      padding: 16px;
    }
    .grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
      gap: 6px;
      padding: 0 16px 16px;
    }
    .grid img {
      width: 100%;
      height: auto;
      display: block;
    }
  </style>
</head>
<body>
  <header>
    <h1>Event Photos</h1>
    <p>Click any photo to view full size.</p>
  </header>
  <main class="grid">
    <a href="FULL_IMAGE_URL"><img src="FULL_IMAGE_URL" alt="Event photo" loading="lazy" decoding="async"></a>
  </main>
</body>
</html>
```

This file is deliberately simple. It is enough to prove the site works before adding all the real photo links.

## Step 3: Commit and push the file

Commit the `index.html` file to the repository’s default branch, usually `main`.

After this step, the repo should contain at least:

- `index.html`
- optionally `README.md`

GitHub Pages needs a valid site entry file at the published location, and `index.html` is the simplest way to provide that.[cite:119][cite:135]

## Step 4: Enable GitHub Pages

In GitHub:

1. Open the `photos` repository.
2. Click **Settings**.
3. In the left sidebar, click **Pages**.
4. Under **Build and deployment**, set **Source** to **Deploy from a branch**.[cite:135]
5. Choose the branch `main`.
6. Choose the folder `/ (root)`.
7. Click **Save**.

GitHub Pages supports publishing a static site directly from a repository branch, which is the simplest option for this tutorial.[cite:89][cite:135]

After a short wait, GitHub should show a default Pages URL for the site.

## Step 5: Add the custom domain

Once the default GitHub Pages site is working:

1. Stay in **Settings → Pages**.
2. In **Custom domain**, enter your domain, for example:

```text
photos.example.org
```

3. Save the custom domain.

GitHub Pages supports custom domains for repository-backed sites and checks DNS before fully enabling the domain.[cite:81][cite:91]

## Step 6: Create the DNS record

In your DNS provider, create a **CNAME** record for the subdomain that points to your GitHub Pages hostname.

Typical example:

- Host/Name: `photos`
- Type: `CNAME`
- Value/Target: `your-org.github.io`

For example, if the GitHub Pages hostname is `exampleorg.github.io`, then `photos.example.org` should point to `exampleorg.github.io`.[cite:82][cite:91]

Do not point the DNS record to the GitHub repo URL like `github.com/exampleorg/photos`. The CNAME must point to the GitHub Pages hostname instead.[cite:82][cite:91]

## Step 7: Wait for GitHub Pages to validate DNS

Go back to **Settings → Pages** and wait for the DNS check to succeed. Once GitHub confirms the DNS is correct, enable **Enforce HTTPS**.[cite:81][cite:104]

At this point, your site should load from the custom domain.

## Step 8: Upload photos to object storage

Upload your event photos into a clean public folder or prefix such as:

```text
public/2026/
```

Consistent folder naming makes it much easier to list files, update the gallery page, and repeat the process next year.

## Step 9: Make the photos public

For the gallery to work, the images must be publicly readable through direct URLs. This can be done with object ACLs or with a bucket policy that grants `GetObject` access to everyone for the event photo bucket or public prefix.[cite:191][cite:168][cite:182]

The simplest operational choice is to use a dedicated public photo bucket or a dedicated public prefix.

Important warning: if the bucket is public, only store public event photos in it.

## Step 10: Test direct image URLs

Before updating the gallery page, open a few direct image URLs in a private browser window.

A typical object URL looks like this:

```text
https://bucket-name.provider-endpoint.example/public/2026/photo-001.jpg
```

If the raw image URL does not work directly, the GitHub Pages site will not be able to load it either.[cite:92]

## Step 11: Add photos to `index.html`

Once a test image works, update the gallery by adding more image links to the `.grid` section.

A single gallery item looks like this:

```html
<a href="https://bucket-name.provider-endpoint.example/public/2026/photo-001.jpg">
  <img
    src="https://bucket-name.provider-endpoint.example/public/2026/photo-001.jpg"
    alt="Event photo"
    loading="lazy"
    decoding="async">
</a>
```

Repeat that structure for all photos.

For a large gallery, it is easier to generate those repeated lines from a list of object keys, but the final published site can still remain a single plain HTML file.

## Step 12: Test the finished gallery

Before sharing the link with attendees, test these things:

- The homepage loads from the custom domain.
- Images render in the grid.
- Clicking a photo opens the full-size image.
- The image can be downloaded in a normal browser flow.
- The gallery works when logged out or in a private browser window.

This last check matters because object storage permissions can appear to work in the console while still failing for normal visitors.

## Recommended simplicity rules

To keep this easy for other organizers to reuse, follow these rules:

- Keep the site in one `index.html` file if possible.
- Keep the site repo separate from the image storage.
- Use GitHub Pages with branch-based publishing instead of adding extra build tools unless necessary.[cite:89][cite:135]
- Use a dedicated public photo bucket or public prefix.
- Use predictable folder naming such as `public/2026/`.
- Avoid app servers, databases, CMS tools, or advanced gallery frameworks.

These choices keep the setup understandable and easy to hand off.

## Tradeoffs to accept

This simple approach has some limits:

- It may not be the fastest possible gallery if full-size images are used directly in the grid.
- It does not include advanced search, tagging, or album management.
- It may require regenerating `index.html` when new photos are added.
- It assumes the photos are meant for public download.

Those tradeoffs are acceptable when the main goal is a low-maintenance, portable public gallery.

## Optional improvements later

Once the simple version is working, teams can improve it without changing the architecture:

- Add smaller thumbnails for better performance.[cite:219][cite:227]
- Add a lightweight lightbox.[cite:223][cite:219]
- Generate the HTML gallery automatically from a list of object keys.[cite:205][cite:211]
- Create multiple pages, such as one gallery per year.

These are optional. They are not required for a successful first version.

## Reuse checklist

Use this checklist for each new event:

1. Create or reuse the `photos` GitHub repo.
2. Confirm GitHub Pages is enabled.[cite:89][cite:135]
3. Confirm the custom domain still points to GitHub Pages.[cite:81][cite:91]
4. Upload event photos to the public bucket or prefix.
5. Confirm direct image URLs work publicly.[cite:92]
6. Update `index.html` with the gallery links.
7. Commit and push changes to `main`.
8. Test the site in a private browser window.
9. Share the gallery link.

## Final guidance

This pattern works because it keeps responsibilities separate:

- GitHub Pages serves the website.
- S3-compatible object storage serves the photos.

That separation keeps the repo lightweight, makes the site easy to version in Git, and avoids turning a simple event photo gallery into a full software project.[cite:208][cite:81][cite:68]
