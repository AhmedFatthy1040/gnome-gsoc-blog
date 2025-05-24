+++
date = '2025-05-23T16:52:16+03:00'
draft = false
title = 'Introduction'
+++

Hello everyone! 👋

I'm Ahmed, a senior Computer Science student at Helwan University. I'm thrilled to be part of **Google Summer of Code 2025** with **GNOME**! This blog post is the first in a series I'll write throughout the summer to share my journey. I'll start with how I got into open source, why I chose GNOME, and what my project is all about.

---


## 🌐 Why Open Source?

My interest in open source began out of frustration, other platforms were too heavy, resource-consuming, or simply didn't respect user privacy. Open source software offered me freedom: to learn, to understand, and to use tools that align with my values.

Contributing to open source is important because it keeps these alternatives alive. These amazing apps don't sustain themselves, they grow because people care enough to contribute. For students like me, open source is also a perfect way to gain real-world experience and build a portfolio that shows initiative and skill. You're not just writing code, you're helping real people.

---

## 📝 My Contributions to GNOME Before GSoC

Before being accepted into GSoC, I had the opportunity to contribute to several GNOME projects, which helped me get familiar with the codebase and the community:


### GNOME Papers

- **Move Document Mutex Locks to Backends** ([MR #499](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/499)):  
  I refactored the code to move mutex lock/unlock operations from the libview and shell components into the document backends (djvu, pdf, tiff, comics). This ensured thread-safe document handling and avoided race conditions.

- **Add 'Open With...' Button to File Properties Dialog** ([MR #526](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/526)):  
  I added an "Open With..." button to the file properties dialog, making it easier for users to open PDFs with other applications (like PDF Arranger or Paper Clip) directly from the properties dialog, instead of searching for the option elsewhere. This improved the discoverability and usability of the feature.

- **Add File Path Display and Copy Buttons to Document Properties Dialog** ([MR #525](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/525)):  
  I implemented a feature to display the full file path in the document properties dialog, along with copy buttons for the file name, folder path, and full file path. This also included handling Flatpak sandboxed paths. The change made it much easier for users to access and copy file paths, especially in sandboxed environments.


### Evolution

- **Fix: Append ".mbox" Extension in Drag-and-Drop Export** ([MR #191](https://gitlab.gnome.org/GNOME/evolution/-/merge_requests/191)):  
  I fixed an issue where emails exported via drag-and-drop did not have the ".mbox" extension, ensuring consistency with the "Save to File..." option. This made exported files more predictable and user-friendly.


### File Roller

- **Show Archive Name in File Chooser Titlebar in Extract Dialog** ([MR #135](https://gitlab.gnome.org/GNOME/file-roller/-/merge_requests/135)):  
  I updated the "Extract File" dialog to display the archive's name in the title bar, helping users remember which archive they are extracting. This small change improved usability, especially when working with multiple archives.

These contributions not only improved my technical skills but also gave me valuable experience collaborating with the GNOME community. They helped me feel prepared and confident to take on a larger project for GSoC.

---

## 🧩 My First Contribution

My first open source contribution was to a game on **Sugarizer**, a learning platform for children. I fixed a bug, a small change, but meaningful. It felt incredible to know that something I wrote could actually improve someone else's experience. That moment got me hooked.

---

## 🐧 Discovering GNOME

I discovered GNOME when I installed **Fedora**, my first Linux distribution. I was immediately drawn to the clean design and usability of the GNOME desktop environment.

What I love most about GNOME is its thoughtful design philosophy. It's minimal yet powerful, designed to help you focus rather than distract you. The developer experience is great, and the community is active and helpful.

I chose GNOME for GSoC because it's a large, well-known organization that millions of people rely on. It's exciting to know that my work will be part of something so impactful.

---

## 🛠️ My GSoC 2025 Project

I'll be working on **Papers: Proof of Concept for Backend Isolation in GNOME Papers**.

Currently, GNOME Papers (formerly Evince) processes all documents in a single application process. This introduces major **security and stability risks**: a crash or vulnerability in one document can take down the whole app.

My project aims to build a **proof-of-concept for backend isolation**, where each document runs in a separate process. To do this, I'll:

- Research isolation models used in projects like Glycin, Chromium and WebKit
- Design an inter-process communication mechanism using **D-Bus** and **shared memory**
- Implement a basic multi-process architecture
- Add sandboxing for document processes to reduce risk

This work will lay the foundation for a more secure and robust document viewer in GNOME.

---

## 🙌 Thanks to My Mentors

I'm very grateful to my mentors **Pablo Correa Gómez**, **Markus Gollnitz**, **Lucas Baudin**, and **Qiu Wenbo** for their guidance and support so far. It's been a pleasure learning from you already, and I'm excited for the summer ahead.

---

That's all for now, I'll be sharing updates every two weeks as I make progress on the project. Stay tuned!

---

## 📬 Connect with Me

- [GitHub](https://github.com/AhmedFatthy1040)
- [GNOME GitLab](https://gitlab.gnome.org/ausername1040)
- [LinkedIn](https://www.linkedin.com/in/ahmedfatthi1040/)

— Ahmed Fatthi Al-Khateeb
