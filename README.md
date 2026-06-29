from pathlib import Path
import hashlib


class DuplicateImageFinder:

    IMAGE_EXTENSIONS = {
        ".jpgg",
        ".jpeg",
        ".png",
        ".gif",
        ".webp",
        ".bmp"
    }

    def __init__(self, folder):
        self.folder = Path(folder)
        self.hashes = {}
        self.duplicates = []

    def calculate_hash(self, file_path):
        hasher = hashlib.sha256()

        with open(file_path, "rb") as f:
            while chunk := f.read(8192):
                hasher.update(chunk)

        return hasher.hexdigest()

    def scan(self):

        print("Scanning images...\n")

        for file in self.folder.rglob("*"):

            if (
                file.is_file()
                and file.suffix.lower()
                in self.IMAGE_EXTENSIONS
            ):

                try:
                    image_hash = self.calculate_hash(file)

                    if image_hash in self.hashes:

                        self.duplicates.append(
                            (
                                self.hashes[image_hash],
                                file
                            )
                        )

                    else:
                        self.hashes[image_hash] = file

                except Exception:
                    pass

    def report(self):

        if not self.duplicates:
            print("No duplicate images found.")
            return

        total_size = 0

        print("\nDuplicate Images Found\n")
        print("=" * 70)

        for original, duplicate in self.duplicates:

            size = duplicate.stat().st_size

            total_size += size

            print(f"\nOriginal : {original}")
            print(f"Duplicate: {duplicate}")
            print(
                f"Size     : "
                f"{size / 1024:.2f} KB"
            )

        print("\n" + "=" * 70)
        print(
            f"Potential space savings: "
            f"{total_size / (1024*1024):.2f} MB"
        )


def main():

    folder = input(
        "Folder path: "
    ).strip()

    finder = DuplicateImageFinder(folder)

    finder.scan()

    finder.report()


if __name__ == "__main__":
    main()
